---
description: Sender constrain refresh tokens in Authgear
---

# Sender Constraining

Sender-constraining is a security mechanism that reduces the risk of access token and refresh token misuse if those tokens are leaked or stolen. Authgear supports sender-constraining using **DPoP (Demonstration of Proof of Possession,** defined in [OAuth 2.0 specification](https://datatracker.ietf.org/doc/html/rfc9449))  for refresh tokens, enabling stronger protection for supported clients.

### What Is Sender-Constraining?

By default, OAuth tokens are **bearer tokens**: anyone who possesses a valid token can use it. If a token is leaked through logging, device compromise, or network interception, an attacker can reuse the token until it expires or is revoked.

Sender-constraining mitigates this risk by **binding a token to a specific client**. Instead of being usable by any party, a sender-constrained token requires the client to cryptographically prove its identity when using the token.

DPoP is a sender-constraining mechanism where the client attaches a signed proof to requests, demonstrating possession of a private key associated with the token.

### How Authgear Uses DPoP

Authgear supports DPoP-based sender-constraining **for refresh tokens**.

When DPoP is used:

* The refresh token is bound to a client-generated cryptographic key
* Refresh token requests must include a valid DPoP proof
* The proof demonstrates that the requester possesses the private key associated with the token

This prevents refresh token reuse by attackers, even if the token value is stolen.

### Supported Clients

DPoP is supported in newer Authgear Mobile SDKs which automatically manage DPoP key generation and storage.

Since secure private key storage is not reliably available in web environments. DPoP is therefore not supported in Authgear Web SDKs

### How to Enable DPoP Validation

You can enable DPoP validation at the **application level** in the Authgear Portal. This option is available only when the application type is **Native App**.

1. Sign in to the **Authgear Portal**
2. Navigate to **Applications**
3. Select the application you want to configure
4. Enable the **DPoP validation** option
5. Save your changes

Once enabled, Authgear will validate DPoP headers for requests from supported clients associated with this application.

### Backward Compatibility

DPoP validation applies only to refresh tokens that were issued with DPoP binding.

Refresh tokens issued without DPoP binding are unaffected and continue to work normally.

This option:

* Does **not** require all clients to send DPoP proofs
* Does **not** block or disable non-DPoP clients
* Does **not** retroactively invalidate existing tokens

