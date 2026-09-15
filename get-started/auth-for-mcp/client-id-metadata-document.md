---
description: >-
  Let OAuth clients, such as the MCP clients used by AI agents, identify
  themselves with a metadata document they host at a URL, instead of registering
  with your project or an admin creating every application by hand.
---

# Client ID Metadata Document (CIMD)

### What is a Client ID Metadata Document? <a href="#what-is-a-client-id-metadata-document" id="what-is-a-client-id-metadata-document"></a>

A Client ID Metadata Document ([CIMD](https://datatracker.ietf.org/doc/draft-ietf-oauth-client-id-metadata-document/02/)) is a JSON document that a client hosts at an HTTPS URL, containing the same metadata it would otherwise send when registering: `redirect_uris`, `client_name`, and so on.

The URL **is** the `client_id`. There is no registration step and no credential to hand out. The client uses its document's URL as its client ID, and Authgear fetches the document the first time it sees that URL in an authorization request.

```
GET /oauth2/authorize
  ?client_id=https://mcp-client.example.com/oauth/client-metadata.json
  ...
```

Clients identified this way are always **third-party public clients**: they use the Authorization Code Flow with PKCE, their users see a consent screen, and no client secret is ever issued.

## When to use CIMD <a href="#when-to-use-cimd" id="when-to-use-cimd"></a>

* **MCP servers and AI agents.** The [MCP Authorization specification](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization) fixes the order in which a client identifies itself: a pre-registered `client_id` if it has one, otherwise CIMD if your authorization server advertises support, and only then [Dynamic Client Registration](dynamic-client-registration.md). Once CIMD is enabled, it is therefore the mechanism a compliant MCP client uses against your project. See [Auth for MCP](README.md) for the full walkthrough.
* **Widely distributed clients.** A desktop app, CLI, or IDE extension shipped to many users identifies itself with one document at one URL, rather than every installation registering a separate client and consuming a slot in your project.
* **Clients whose metadata you do not want to maintain.** Authgear refetches the document once an hour, so the stored metadata follows the client's own document, and no admin has to update it.

**When not to use it:** for your own fixed set of apps, create them under **Applications** → **Client Applications** as usual. For backend services calling your APIs with their own credentials, use [Machine-to-Machine (M2M) Applications](../m2m-applications.md). For clients that cannot host a document but can call an endpoint, use [DCR](dynamic-client-registration.md).

## How CIMD compares to DCR <a href="#how-cimd-compares-to-dcr" id="how-cimd-compares-to-dcr"></a>

Both let clients reach your project without an admin creating them, and both appear in the same client list. They differ in who holds the identity:

|                        | CIMD                                        | DCR                                            |
| ---------------------- | ------------------------------------------- | ---------------------------------------------- |
| `client_id`            | The document's URL                          | Issued by Authgear (`dcrc_…`)                  |
| Who may get in         | **Trusted domains** (any domain by default) | **Initial access token** (required by default) |
| Metadata freshness     | Refetched hourly from the document          | Frozen at registration                         |
| Changing the client ID | Move the document, and it is a new client   | The `client_id` is stable                      |
| Client kind            | Always third-party                          | First- or third-party, depending on the token  |

The two can be enabled at the same time; a client uses whichever it supports.

## Enable CIMD in the Portal <a href="#enable-cimd-in-the-portal" id="enable-cimd-in-the-portal"></a>

### 1. Turn on CIMD

Go to **Applications** → **AI Agents (Dynamic)** → the **CIMD** tab, and turn on **Enable CIMD**. The switch saves immediately.

Your project's discovery documents then advertise `client_id_metadata_document_supported: true`, which tells a compliant client it may use its document URL as a `client_id`.

### 2. Choose which domains you trust

On the **Trusted domains** card, pick one and click **Save**:

* **Any domain**: any app that publishes a valid metadata document can sign your users in. This is the default, and it is what the MCP use case needs, because you cannot know in advance which AI agents your users will bring. Switching back to it asks you to confirm.
* **Only specific domains**: only apps whose document is hosted on a listed domain can sign in. Enter a hostname such as `mcp.example.com`, or a wildcard such as `*.example.com`. A wildcard matches exactly one label: `a.example.com`, but not `a.b.example.com`, and not `example.com` itself.

{% hint style="info" %}
Trusted domains decides who may **onboard**. It is checked when a document is fetched, so removing a domain stops new clients from that domain and stops refetches of existing ones. A client already resolved keeps working with its last-fetched metadata until you delete it from the client list.
{% endhint %}

### 3. Optional: client configuration

The **Client configuration** card sets the token lifetimes (access token, refresh token, refresh token idle timeout) applied to every client resolved from a metadata document. Click **Save** after changing them. A document cannot set these itself, and they cannot be set per client.

### 4. Grant access to your APIs

This step is **required** if your clients call your APIs, and an MCP client always does. CIMD clients are dynamic third-party clients, so they can only reach API resources you explicitly open to them:

* On an API resource's detail page (**API Resources** section), turn on **Allow dynamic third-party clients**.
* On each scope those clients should be able to request, check **Allow dynamic third-party clients to request this scope**.

Without this, an authorization request naming your API with the `resource` parameter fails with `invalid_target`, however valid the client's document is. All dynamic clients share this access, and your first-party clients are unaffected. The **Overview** tab's **Allowed API resources** card shows what is currently open.

## Host the document <a href="#host-the-document" id="host-the-document"></a>

This part lives with the client, not in Authgear. Serve a JSON document at the URL the client uses as its `client_id`:

```json
{
  "client_id": "https://mcp-client.example.com/oauth/client-metadata.json",
  "client_name": "Example MCP Client",
  "redirect_uris": [
    "http://127.0.0.1:3000/callback",
    "http://localhost:3000/callback"
  ],
  "grant_types": ["authorization_code", "refresh_token"],
  "response_types": ["code"],
  "token_endpoint_auth_method": "none"
}
```

Requirements on the `client_id` URL:

* `https` scheme, with a path. `https://example.com` alone is not a valid client ID; `https://example.com/client.json` is.
* No fragment, and no username or password.
* The `client_id` **inside** the document must equal the URL byte for byte. A trailing slash or a difference in case is a mismatch.

Rules for the metadata fields:

* `redirect_uris` is required. Each entry must be `https://`, a loopback `http://` URI (`localhost`, `127.0.0.1`, or `[::1]`, on any port), or a custom scheme such as `com.example.app:/callback`.
* `grant_types` may contain only `authorization_code` and `refresh_token`, and `response_types` only `code`. Both default to those values when omitted.
* `token_endpoint_auth_method` must be `none` if present.
* `logo_uri`, `client_uri`, `tos_uri`, and `policy_uri` must be `https://`.
* Anything else, including `client_secret`, is ignored. A document that breaks any rule above is treated as if the fetch had failed.

How Authgear fetches it:

* With a plain `GET`. Redirects are not followed, so the URL must serve the document directly.
* The response is limited to 5 KB.
* The document is refetched once an hour. A change therefore takes up to an hour to take effect for clients Authgear has already seen.

If you set `logo_uri`, the image must be PNG, JPEG, GIF, or WebP, and at most 256 KiB. SVG is refused.

## Manage resolved clients <a href="#manage-resolved-clients" id="manage-resolved-clients"></a>

CIMD clients appear alongside DCR ones under **Applications** → **AI Agents (Dynamic)** → **Overview** → **View all**. A **Type** column and filter distinguish them, and a **Last fetched** column shows document freshness.

* **Deleting a CIMD client is not a block.** It clears the metadata Authgear stored, but the same `client_id` is resolved again the next time someone signs in with it. To keep a client out for good, set **Trusted domains** to **Only specific domains** without its domain *and* delete it.
* **Client limit:** your plan may cap how many CIMD clients your project holds at once, counted separately from DCR clients. At the cap, resolving a *new* client ID fails with `access_denied`; clients already resolved keep working and keep refetching.
* **Turning CIMD off** stops new clients being resolved and stops refetches. Clients already resolved keep working. Delete them if you want their access gone.
* **Audit log.** `oauth.client.resolved` is recorded when a client is first resolved or its metadata changes, and `oauth.client.resolution.failed` when a document is unreachable or invalid, or the client limit is hit. A client refused because CIMD is off or its domain is not trusted is not logged.

## Errors a client can see <a href="#errors-a-client-can-see" id="errors-a-client-can-see"></a>

| Situation                                                                                                          | Error from `/oauth2/authorize`                                                          |
| ------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------- |
| The document is unreachable or invalid, the domain is not trusted, or CIMD is off, and Authgear has no stored copy | `unauthorized_client` with the message "invalid client ID", the same as for an unknown `client_id` |
| `resource` names an API not opened to dynamic third-party clients                                                  | `invalid_target`                                                                        |

## Security notes <a href="#security-notes" id="security-notes"></a>

Fetching a document means making an outbound request to a URL a stranger chose, so Authgear constrains it. Fetches go only to public internet addresses, follow no redirects, and are capped in size, time, and rate. Every failure returns the same error, so nobody can use your project to probe which hosts its network can reach. With **Trusted domains** set to specific domains, client IDs on any other host are refused before any fetch.

## Try it end to end <a href="#try-it-end-to-end" id="try-it-end-to-end"></a>

Real clients do this automatically, but you can walk the flow by hand. Host a document as above, then start an authorization request with its URL as the `client_id`:

```
GET https://<your-project>.authgear.cloud/oauth2/authorize
  ?client_id=https://mcp-client.example.com/oauth/client-metadata.json
  &response_type=code
  &scope=openid+read:tools
  &redirect_uri=http://127.0.0.1:3000/callback
  &code_challenge=<challenge>
  &code_challenge_method=S256
  &resource=https://mcp-server.example.com
```

Authgear fetches the document, validates `redirect_uri` against it, and shows the consent screen labeled with `mcp-client.example.com`. Exchange the code at `/oauth2/token` with the same `code_verifier`, `redirect_uri`, and `resource`, and the access token's `aud` claim is your API's URI, exactly what your resource server validates.
