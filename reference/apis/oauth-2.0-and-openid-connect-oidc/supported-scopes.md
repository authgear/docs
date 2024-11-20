---
description: Details about the OAuth 2.0 Scopes Authgear supports
---

# Supported Scopes

Scope is a security mechanism that controls the level of access a client application can have to a user's data. Scope is part of the [OAuth 2.0 specification](https://datatracker.ietf.org/doc/html/rfc6749#section-3.3) and the value of scope is a parameter that contains a list of one or more scopes. Each scope is separated from the other using a single white space.&#x20;

The following example shows the structure of the Authgear authorization endpoint with `openid` and `offline_access` scopes:

```
GET https://your-project.authgear.cloud/oauth2/authorize?
    client_id=<CLIENT_ID>&
    scope=openid+offline_access&
    redirect_url=<REDIRECT_URI>
```

In this post, we'll discuss all the OIDC scopes Authgear supports and the level of access each scope grants a client application.

### 1. openid

This is the minimum scope required to use Authgear as an OIDC or 0Auth 2.0 provider. Usually, you'll use the `openid` scope alone or with other additional scopes based on the kind of data your client application will access from Authgear on behalf of a user.

The `openid` scope grants the following access to your client application:

* **Open ID Connect:** required to access the Authgear authorization page, get the authorization code, and exchange the authorization code for an access token.
* **UserInfo Endpoint:** You can access a user's profile details such as email, phone number, picture, gender and more using the access token gotten from an authorization request that has the `openid` scope in a separate HTTP request to the [UserInfo](https://docs.authgear.com/reference/apis/oauth-2.0-and-openid-connect-oidc/userinfo) endpoint.

The following is a sample of the HTTP response from the token endpoint for an authorization request that has only the `openid` scope:

```json
{
  "access_token": "<ACCESS_TOKEN_VALUE>",
  "expires_in": 1800,
  "id_token": "<ID_TOKEN_VALUE>",
  "token_type": "Bearer"
}
```

### 2. offline\_access

The `offline_access` scope is required to get a **refresh token** from Authgear's token endpoint. If a client application that does not include the `offline_access` scope in its authorization request exchanges an authorization code for an access token, the `refresh_token` field will be omitted.

The refresh token is a key that your client application can send back to the Authgear server for a new access token after the old access token expires. Hence it allows your application to obtain a new access token without the user repeating the entire OAuth 2.0 authorization flow.

The following example shows the response from the token endpoint for an authorization request with the `offline_access` scope:

```json
{
  "access_token": "<ACCESS_TOKEN_VALUE>",
  "expires_in": 1800,
  "id_token": "<ID_TOKEN_VALUE>",
  "refresh_token": "<REFRESH TOKEN_VALUE>",
  "token_type": "Bearer"
}
```

### 3. https://authgear.com/scopes/full-userinfo

Including the  `https://authgear.com/scopes/full-userinfo` scope in an authorization request will enable Authgear to include some of the details that are usually returned by the UserInfo endpoint in the ID token for the current session. Details such as the email address, phone number, and account verification status are included.

The value for `id_token` is an encoded string, you'll need to decode this value to view the actual user profile information.

Although the `id_token` field is present in the token endpoint response for authorization requests that does **NOT** include the `https://authgear.com/scopes/full-userinfo` scope, decoding the `id_token` in such responses will not include user profile information.

The following example shows the decoded content of an id\_token for an authorization request with the `https://authgear.com/scopes/full-userinfo` scope present:

```json
{
  "aud": [
    "<CLIENT_ID>"
  ],
  "auth_time": 1714667006,
  "email": "johndoe@example.com",
  "email_verified": false,
  "exp": 1714667852,
  "https://authgear.com/claims/user/can_reauthenticate": true,
  "https://authgear.com/claims/user/is_anonymous": false,
  "https://authgear.com/claims/user/is_verified": false,
  "https://authgear.com/claims/user/roles": [],
  "iat": 1714667552,
  "iss": "https://authgeartest.online",
  "sid": "<SID_VALUE>",
  "sub": "<SUB_VALUE>"
}
```

Here's an example of the decoded content of an id\_token for an authentication request that does NOT include the `https://authgear.com/scopes/full-userinfo` scope:

```json
{
  "aud": [
    "<CLIENT_ID>"
  ],
  "auth_time": 1714667006,
  "exp": 1714668328,
  "https://authgear.com/claims/user/can_reauthenticate": true,
  "https://authgear.com/claims/user/is_anonymous": false,
  "https://authgear.com/claims/user/is_verified": false,
  "https://authgear.com/claims/user/roles": [],
  "iat": 1714668028,
  "iss": "https://authgeartest.online",
  "sid": "<SID_VALUE>",
  "sub": "<SUB_VALUE>"
}
```

Notice that the `email` and `email_verified` fields are not included in the second example.

