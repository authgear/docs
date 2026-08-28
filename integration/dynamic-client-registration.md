---
description: >-
  Let OAuth clients, such as the MCP clients used by AI agents, register themselves with
  your project at runtime using Dynamic Client Registration (RFC 7591), instead
  of an admin creating every application by hand.
---

# Dynamic Client Registration (DCR)

### What is Dynamic Client Registration? <a href="#what-is-dynamic-client-registration" id="what-is-dynamic-client-registration"></a>

Normally, every application that talks to your Authgear project is created by an admin in the Portal. Dynamic Client Registration (DCR, [RFC 7591](https://datatracker.ietf.org/doc/html/rfc7591)) removes that manual step: when enabled, your project exposes a registration endpoint at `/oauth2/register`, and clients create themselves at runtime by sending their own metadata.

Clients registered this way are **public clients**: they authenticate users with the Authorization Code Flow with PKCE and never receive a client secret.

## When to use DCR <a href="#when-to-use-dcr" id="when-to-use-dcr"></a>

* **MCP servers and AI agents:** The [MCP Authorization specification](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization) expects the authorization server to support DCR: each user's MCP client (an AI assistant, an IDE agent) registers itself with your project at first use, with no admin involvement per client. This is the main reason to turn DCR on; see [Auth for MCP](../get-started/auth-for-mcp.md) for the full walkthrough.
* **Third-party developer ecosystems:** Let partners and external developers building integrations against your APIs register their own OAuth clients. You hand each developer an initial access token instead of creating clients for them; their users see a consent screen before granting access.
* **Ephemeral and automated environments:** A CI pipeline can register a short-lived client for every preview deployment, so each environment gets its own client ID without anyone touching the Portal.

**When not to use it:** for your own fixed set of apps, create them in the Portal's **Applications** page as usual; for backend services calling your APIs with their own credentials, use [Machine-to-Machine (M2M) Applications](../get-started/m2m-applications.md).

## Key concepts <a href="#key-concepts" id="key-concepts"></a>

* **Registration endpoint:** `https://<your-project>.authgear.cloud/oauth2/register`. While DCR is enabled it is also advertised as `registration_endpoint` in your project's OpenID Connect discovery document.
* **Initial access token (IAT):** By default, callers must present a valid IAT as a Bearer token to register. You create IATs in the Portal and control who can register by controlling who holds a token.
* **Token type (third-party vs first-party):** A **third-party** IAT registers clients whose users see a consent screen; it is safe to hand to external developers. A **first-party** IAT registers clients that skip the consent screen; treat it like your Admin API key.
* **Open registration:** Turning the IAT requirement off lets anyone register a client with your project. This is what the MCP flow expects; note that your plan's client limit bounds how many clients can accumulate.

## Enable DCR in the Portal <a href="#enable-dcr-in-the-portal" id="enable-dcr-in-the-portal"></a>

### 1. Turn on registration

In the Portal, go to **Applications** → the **Dynamic clients (DCR)** tab. Turn on **Enable dynamic client registration** and click **Save**. The tab then shows your project's registration endpoint.

### 2. Decide how registration is protected

Under **Registration security**, the **Require initial access token** toggle is on by default:

* Keep it on and click **Create token** to mint an IAT. Pick the token type (**Third-party** or **First-party**) and an expiry. The token value is shown once, together with a ready-to-run example request; copy the token and store it securely.
* Or turn the requirement off to allow **open registration** (e.g. for MCP). The Portal asks you to confirm, since anyone will be able to register clients with your project.

{% hint style="warning" %}
A first-party initial access token registers clients that skip the consent screen. Only use it for automation you fully control, and protect it like the Admin API private key.
{% endhint %}

### 3. Optional: default client configuration

The **Default client configuration** card sets the token lifetimes (access token, refresh token, refresh token idle timeout) applied to every dynamically registered client, in case these clients need stricter settings than your own apps do.

### 4. Grant access to your APIs

Dynamic **third-party** clients can only call API resources you explicitly open to them:

* On an API resource's detail page (**API Resources** section), turn on **Allow dynamic third-party clients**.
* On each scope of that resource, check **Allow dynamic third-party clients to request this scope**.

All dynamic third-party clients share this access; your first-party clients are unaffected.

## Register a client <a href="#register-a-client" id="register-a-client"></a>

{% tabs %}
{% tab title="With an initial access token" %}
```bash
curl https://<your-project>.authgear.cloud/oauth2/register \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <your-initial-access-token>' \
  -d '{
    "client_name": "My integration",
    "redirect_uris": ["https://myintegration.example.com/callback"],
    "application_type": "web"
  }'
```
{% endtab %}

{% tab title="Open registration" %}
```bash
curl https://<your-project>.authgear.cloud/oauth2/register \
  -H 'Content-Type: application/json' \
  -d '{
    "redirect_uris": ["https://mcp-client.example.com/callback"]
  }'
```
{% endtab %}
{% endtabs %}

A successful registration returns **201 Created** with the new client's metadata:

```json
{
  "client_id": "dcrc_AbCdEfGhIjKlMnOpQr",
  "client_id_issued_at": 1700000000,
  "client_name": "My integration",
  "redirect_uris": ["https://myintegration.example.com/callback"],
  "grant_types": ["authorization_code", "refresh_token"],
  "response_types": ["code"],
  "application_type": "web"
}
```

Rules to keep in mind:

* `application_type` is `web` (the default) or `native`. A `web` client's redirect URIs must use `https`; a `native` client may use custom schemes (e.g. `myapp://callback`) or `http://localhost`.
* No client secret is ever issued: `token_endpoint_auth_method` is always `none`, and requests asking for anything else are rejected.
* Whether the client is first-party or third-party is decided by the IAT used to register it; open registration always registers third-party clients.

## Use the registered client <a href="#use-the-registered-client" id="use-the-registered-client"></a>

The client then runs the standard Authorization Code Flow with PKCE, passing the `resource` parameter to name the API it wants to call:

```
GET /oauth2/authorize
  ?client_id=dcrc_AbCdEfGhIjKlMnOpQr
  &response_type=code
  &scope=openid+read:tools
  &redirect_uri=https://mcp-client.example.com/callback
  &code_challenge=<challenge>
  &code_challenge_method=S256
  &resource=https://mcp-server.example.com
```

The user signs in and, for a third-party client, approves the requested scopes on the consent screen. The client exchanges the code at `/oauth2/token` (including the same `code_verifier`, `redirect_uri` and `resource`), and the issued access token carries the resource URI in its `aud` claim, which your API validates.

## Manage registered clients <a href="#manage-registered-clients" id="manage-registered-clients"></a>

* The **Dynamic clients (DCR)** tab shows how many clients are registered; **View all** opens the full list, where you can inspect each client's metadata and delete it.
* **Client limit:** your plan may cap the number of dynamic clients. Once the cap is reached, further registrations fail with `403 access_denied` until an admin deletes clients.
* **Deleting a client** stops new authorizations immediately; access and refresh tokens already issued stay valid until they expire.
* **Revoking an IAT** stops it from registering new clients; clients already registered with it are unaffected.
* **Disabling DCR later** only closes the registration endpoint; already-registered clients keep working. Delete them if you want their access gone.
