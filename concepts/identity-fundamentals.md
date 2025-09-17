# Identity Fundamentals

### Identity and access management <a href="#what-is-identity-and-access-management-iam" id="what-is-identity-and-access-management-iam"></a>

Identity and Access Management (IAM) is a framework that manages digital identities and their access to various resources within a network or systems. IAM technologies and practices enable the right individuals to access the right resources at the right times and for the right reasons. For example, if a user wants to access a resource, IAM management verifies the user and controls their access to the resources.

Authgear acts as an IAM provider that is a **gatekeeper to the resources** you provide to customers as web and mobile applications, APIs, etc. The gatekeeper initiates authorization as outlined in [OAuth 2.0](identity-fundamentals.md#oauth-2.0). The addition of the [OpenID Connect](identity-fundamentals.md#open-id-connect) layer adds authentication to secure your users’ digital identities and your product.

### Relying party

In the context of identity and access management, a "relying party" is a system or application that relies on a separate system such as **Authgear client SDK or libraries** to authenticate a user's identity. Essentially, the relying party trusts another system to tell it that a user is who they claim to be.

Here's an example to illustrate the concept:

1. A user wants to log into an online service (the relying party).
2. The relying party doesn't handle authentication directly but instead relies on a trusted third-party system like Authgear.
3. The user logs into the third-party system, which verifies their identity.
4. The third-party system tells the relying party that the user's identity has been verified.
5. The relying party then grants the user access based on this verification.

### OAuth 2.0

The [OAuth 2.0](https://tools.ietf.org/html/rfc6749) authorization framework is a protocol that allows a user to grant a third-party website or application access to the user's protected resources, without necessarily revealing their long-term credentials or even their identity. It’s the standard that allows third-party developers to rely on large social platforms like Facebook, Google, and Twitter for login.  Authgear generates access tokens for API authorization scenarios, in [JSON web token](identity-fundamentals.md#json-web-tokens) (JWT) format. The permissions are represented by [the access token](../api-reference/tokens/jwt-access-token.md).

### Open ID Connect

A simple identity layer that sits on top of OAuth 2.0, [OpenID Connect (OIDC)](https://openid.net/developers/how-connect-works/) makes it easy to verify a user’s identity and obtain basic profile information from the identity provider. OIDC is another open standard protocol. You can Authgear **as an Identity Provider (IdP)** that uses  OIDC standards.

### JSON web tokens

[JSON web tokens](https://datatracker.ietf.org/doc/html/rfc7519) (JWTs) are an open standard that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. JWTs can be verified and trusted because they’re digitally signed. They can be used to pass the identity of authenticated users between the identity provider and the service requesting the authentication. They also can be authenticated and encrypted.

