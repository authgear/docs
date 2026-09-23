---
description: List of OAuth error codes returned by the Authgear authorization and token endpoints.
---

# Error Codes

When a request to the authorization endpoint (`/oauth2/authorize`) or the token endpoint (`/oauth2/token`) fails, Authgear returns an OAuth error response as defined in [RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749#section-5.2).

For the token endpoint, the error is returned in the JSON response body:

```json
{
  "error": "invalid_grant",
  "error_description": "invalid authorization code"
}
```

For the authorization endpoint, the error is returned as query parameters on the redirect URI:

```
https://your-app.example.com/callback?error=access_denied&error_description=authorization+denied
```

Use the `error` field to decide how to handle the error. The `error_description` field is meant for developers and may change without notice, so do not match on it in your code.

## Errors in the Authgear SDKs

The Authgear SDKs turn these responses into an `OAuthError`. For example, in the JavaScript, React Native, and Capacitor SDKs:

```typescript
import { OAuthError, CancelError } from "@authgear/capacitor";

try {
  await authgear.authenticate({ redirectURI });
} catch (e) {
  if (e instanceof CancelError) {
    // The user closed the login page.
  } else if (e instanceof OAuthError) {
    // e.error is one of the error codes below.
    console.error(e.error, e.error_description);
  } else {
    // Other errors, such as network errors.
  }
}
```

After the login page closes, the SDK exchanges the authorization code for tokens. If that request fails, the login page has already closed but `authenticate()` rejects with an `OAuthError`. Always handle the rejection. Otherwise, the user sees the login page close and stays logged out without any message.

## Error code list

| `error` | Meaning | Common causes |
| --- | --- | --- |
| `invalid_request` | The request is missing a parameter or has an invalid one. | Missing `code`, `redirect_uri`, `code_verifier`, or `refresh_token`. The redirect URI is not in the **Authorized Redirect URIs** of the application. PKCE is missing or does not use `S256`. The OAuth session expired before the user finished logging in. |
| `invalid_client` | The client could not be authenticated. | Wrong client ID. A missing or wrong client secret for a confidential client. |
| `unauthorized_client` | The client is not allowed to perform this request. | The grant type or response type is not allowed for this application. Anonymous users or biometric login are disabled for the project. |
| `invalid_grant` | The authorization code, refresh token, or other grant is invalid. | The authorization code expired or was already used. The refresh token was revoked or expired. The session ended. |
| `invalid_scope` | The requested scopes are not allowed. | The `openid` scope is missing. The application is not allowed to request `offline_access`, full access, or `device_sso`. |
| `invalid_target` | The requested `resource` is invalid. | The resource does not exist, or the application is not associated with it. |
| `invalid_dpop_proof` | The DPoP proof is invalid or does not match the token. | The device clock is wrong. The DPoP key stored on the device no longer matches the key the token was bound to. See [DPoP errors](#dpop-errors). |
| `access_denied` | The request was denied. | The user denied authorization. The requested scopes were not granted. |
| `login_required` | The user must log in. | `prompt=none` was used but the user has no active session. |
| `insufficient_scope` | The token does not have the scope required for this operation. | The session does not have the scopes needed to create a pre-authenticated URL. |
| `unsupported_grant_type` | The grant type is not supported. | A typo in `grant_type`. |
| `unsupported_response_type` | The response type is not supported. | A typo in `response_type`. |
| `x_rate_limited` | Too many requests. | See [Rate Limits](../../rate-limits/README.md). Retry later. |
| `server_error` | An unexpected error occurred on the server. | Retry later. If the error persists, contact Authgear support. |

## DPoP errors

The Authgear mobile SDKs attach a DPoP proof to every token request. When [DPoP validation](../../../security/sender-constraining.md) is enabled for an application and the proof cannot be verified, the token endpoint returns:

```json
{
  "error": "invalid_dpop_proof",
  "error_description": "Invalid DPoP key binding"
}
```

The most common cause is a wrong device clock. Each DPoP proof contains the time it was created, taken from the device clock. Authgear accepts a clock difference of up to 5 minutes. If the device clock is more than 5 minutes ahead of or behind the actual time, every DPoP proof is rejected, so the user cannot log in or refresh their session.

In this case, the login page closes normally, but the code exchange fails and `authenticate()` rejects with `invalid_dpop_proof`. Ask the user to turn on automatic date and time in the device settings and try again.
