---
description: >-
  Use Authgear as the OAuth authorization server for your MCP (Model Context
  Protocol) server, so AI agents can sign your users in and call your MCP tools
  with properly scoped, audience-bound access tokens.
---

# Auth for MCP

### How MCP authorization works <a href="#how-mcp-authorization-works" id="how-mcp-authorization-works"></a>

The [MCP Authorization specification](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization) is built on standard OAuth 2.0, with the roles split like this:

* Your **MCP server** is an OAuth **resource server**. It never handles passwords; it only validates access tokens.
* **Authgear** is the **authorization server**. It signs users in, shows the consent screen, and issues tokens.
* Each user's **MCP client** (an AI assistant such as Claude, or an IDE agent) is an OAuth client that discovers your authorization server and **identifies itself** at first use, either with a [Client ID Metadata Document](../integration/client-id-metadata-document.md) it hosts or by registering via [Dynamic Client Registration](../integration/dynamic-client-registration.md).

The end-to-end flow, fully automatic once you finish the setup below:

1. The MCP client calls your MCP server without a token and receives `401 Unauthorized` with a `WWW-Authenticate` header pointing at the server's Protected Resource Metadata ([RFC 9728](https://datatracker.ietf.org/doc/html/rfc9728)).
2. That metadata names your Authgear project as the authorization server, and the client fetches Authgear's discovery document.
3. The client identifies itself, with its Client ID Metadata Document if the discovery document advertises `client_id_metadata_document_supported`, otherwise by registering through DCR.
4. The client runs the Authorization Code Flow with PKCE, passing `resource=<your MCP server URI>`. The user signs in with Authgear and approves the requested scopes on the consent screen.
5. Authgear issues an access token whose `aud` claim is your MCP server's URI. The client retries the MCP request with the token; your server validates it and serves the tools.

## Set up Authgear for your MCP server <a href="#set-up-authgear-for-your-mcp-server" id="set-up-authgear-for-your-mcp-server"></a>

### 1. Register the MCP server as an API Resource

In the Portal, go to **API Resources** and create a resource for your MCP server:

* Set the **identifier** to the MCP server's canonical URI, for example `https://mcp-server.example.com`. This is the value MCP clients send as `resource`, and the `aud` your server will validate.
* Add the **scopes** your tools need, e.g. `read:tools` and `execute:tools`.

### 2. Open the resource to dynamic clients

MCP clients are dynamic third-party clients, so grant them access explicitly:

* On the resource's detail page, turn on **Allow dynamic third-party clients**.
* On each scope MCP clients should be able to request, check **Allow dynamic third-party clients to request this scope**.

### 3. Let MCP clients identify themselves

Go to **Applications** → **AI Agents (Dynamic)**. Enable at least one mechanism; enabling both covers every compliant client.

{% tabs %}
{% tab title="CIMD (recommended)" %}
On the **CIMD** tab, turn on **Enable CIMD**.

That is the whole setup. **Trusted domains** stays on **Any domain**, which is what this use case needs, because you cannot know in advance which AI agents your users will bring. No credential has to be distributed to anyone.

See [Client ID Metadata Document (CIMD)](../integration/client-id-metadata-document.md) for the document format and the trust controls.
{% endtab %}

{% tab title="DCR (fallback)" %}
For clients that do not support CIMD, go to the **DCR** tab:

* Turn on **Enable DCR**.
* Under **Registration security**, turn off **Require initial access token**. Generic MCP clients have no way to obtain or present one; with the requirement on, their registration attempts fail with `403 access_denied`.

{% hint style="info" %}
Open registration means anyone can register a client with your project. A registered client can do nothing until a real user signs in and consents, and it can only request the resources and scopes you opened in step 2.
{% endhint %}
{% endtab %}
{% endtabs %}

## Set up the MCP server side <a href="#set-up-the-mcp-server-side" id="set-up-the-mcp-server-side"></a>

Two things live on your MCP server, not in Authgear:

**Serve Protected Resource Metadata.** Host a document at `/.well-known/oauth-protected-resource` naming your Authgear project as the authorization server, and reference it from every `401` response:

```json
{
  "resource": "https://mcp-server.example.com",
  "authorization_servers": ["https://myproject.authgear.cloud"],
  "scopes_supported": ["read:tools", "execute:tools"]
}
```

```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer resource_metadata="https://mcp-server.example.com/.well-known/oauth-protected-resource"
```

**Validate the access token** on every MCP request:

1. Confirm the token is a JWT.
2. Fetch `jwks_uri` from `https://myproject.authgear.cloud/.well-known/openid-configuration` and verify the signature.
3. Check `iss` equals your Authgear project endpoint.
4. Check `aud` includes your MCP server's URI (`https://mcp-server.example.com`). This check stops a token issued for another audience from being replayed against your server.
5. Check the token has not expired (`exp`), and that its `scope` covers the requested tool.

See [Validate JWT in your backend](backend-api/jwt.md) for language-specific examples of steps 1–5.

## Try it end to end <a href="#try-it-end-to-end" id="try-it-end-to-end"></a>

Real MCP clients do all of this automatically, but you can walk the flow by hand to verify the setup.

{% tabs %}
{% tab title="CIMD" %}
Host a metadata document, then use its URL as the `client_id`:

```
GET https://myproject.authgear.cloud/oauth2/authorize
  ?client_id=https://mcp-client.example.com/oauth/client-metadata.json
  &response_type=code
  &scope=openid+read:tools
  &redirect_uri=http://127.0.0.1:3000/callback
  &code_challenge=<challenge>
  &code_challenge_method=S256
  &resource=https://mcp-server.example.com
```
{% endtab %}

{% tab title="DCR" %}
Register a client. A desktop or CLI client redirects to `localhost`, so it must register as `native`, or the request fails with `invalid_redirect_uri`:

```bash
curl https://myproject.authgear.cloud/oauth2/register \
  -H 'Content-Type: application/json' \
  -d '{"redirect_uris": ["http://localhost:3000/callback"], "application_type": "native"}'
```

Then run the Authorization Code Flow with PKCE using the returned `client_id`:

```
GET https://myproject.authgear.cloud/oauth2/authorize
  ?client_id=dcrc_AbCdEfGhIjKlMnOpQr
  &response_type=code
  &scope=openid+read:tools
  &redirect_uri=http://localhost:3000/callback
  &code_challenge=<challenge>
  &code_challenge_method=S256
  &resource=https://mcp-server.example.com
```
{% endtab %}
{% endtabs %}

After sign-in and consent, exchange the code at `/oauth2/token` (with the same `code_verifier`, `redirect_uri` and `resource`). Decode the resulting access token and confirm `aud` is `["https://mcp-server.example.com"]`, exactly what your MCP server validates.

## Operating notes <a href="#operating-notes" id="operating-notes"></a>

* Every MCP client that identified itself appears under **Applications** → **AI Agents (Dynamic)** → **Overview** → **View all**, with a **Type** column showing whether it came in via CIMD or DCR. From there you can inspect its metadata or delete it.
* Users manage consent for MCP clients the same way as for any other third-party app; the consent screen shows the scopes the MCP client requested, and for a CIMD client it also shows the hostname its document is served from.
* Token lifetimes are set in the **Client configuration** card on each mechanism's tab.
