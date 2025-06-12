---
description: Integrate Authgear on the client side in mobile apps without SDK
---

# Using Authgear without SDK (Client side)

[![LLM | View as markdown](https://img.shields.io/badge/LLM-View%20as%20markdown-blue)](https://raw.githubusercontent.com/authgear/docs/refs/heads/main/get-started/native-mobile-app/using-authgear-without-sdk-client-side.md)

Authgear is built based on OIDC 2.0 standard, you can integrate it into any application framework without SDK.

### Supported grant types

**Authorization Code Flow with Proof Key for Code Exchange (PKCE)**

For public clients like mobile apps and SPAs, PKCE flow should be used. It uses `code_challenge_method` & `code_challenge` parameters in the authentication requests and the code\_verifier parameter in the code exchange step. It ensures that the application that starts the code flow is the same one that finishes it.

### OpenID Connect Configurations

This endpoint serves as a JSON document containing the OpenID Connect configuration of your Authgear project. That includes the authorization endpoint, the token endpoint, and the JWKs endpoint. The URL looks like:

```
https://AUTHGEAR_ENDPOINT/.well-known/openid-configuration
```

### Initiate Signup and Login

To singup or login, redirect the user to the `/authorize` endpoint. The URL should look like this:

```url
https://AUTHGEAR_ENDPOINT/oauth2/authorize
?response_type=code
&client_id=CLIENT_ID
&redirect_uri=REDIRECT_URL
&scope=openid+offline_access+https://authgear.com/scopes/full-userinfo
&code_challenge_method=S256
&code_challenge=CODE_CHALLENGE
&state=abc
```

**Code Example**

{% tabs %}
{% tab title="curl" %}
```bash
curl -X GET 'https://AUTHGEAR_ENDPOINT/oauth2/authorize' \
--data-urlencode 'response_type=code' \
--data-urlencode 'client_id=CLIENT_ID' \
--data-urlencode 'redirect_uri=REDIRECT_URL' \
--data-urlencode 'scope=openid offline_access https://authgear.com/scopes/full-userinfo' \
--data-urlencode 'code_challenge_method=S256' \
--data-urlencode 'code_challenge=CODE_CHALLENGE' \
--data-urlencode 'state=abc'
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const randomData = secureRandomBytes(32); // (1)
const code_verifier = base64url(randomData); // (2) 
const hash = await sha256(utf8(code_verifier));
const code_challenge = base64url(hash);

// (1) A 32-byte long secure random data using secure random number generator
// (2) this string must be between 43 and 126 bytes long

const params = new URLSearchParams();
params.set("response_type", "code");
params.set("client_id", "CLIENT_ID");
params.set("redirect_uri", "REDIRECT_URL");
params.set("scope", ["openid", "offline_access", "https://authgear.com/scopes/full-userinfo"].join(" "));
params.set("code_challenge_method", "S256");
params.set("code_challenge", code_verifier);
params.set("state", "abc");

const endpoint = new URL("https://AUTHGEAR_ENDPOINT/oauth2/authorize");
const authorizationURL = endpoint.toString() + "?" + params.toString();
```
{% endtab %}
{% endtabs %}

* “S256” is supported in code\_challenge\_method.
* The `code_verifier` is a string of length between 43 and 128 generated on the client side. It should be unique for each authorization request. The `code_challenge` is a SHA-256 hash of the verifier.
* The state parameter is optional. Authgear will include the value of the state parameter when redirecting the user back to the client application. Learn more at [how-to-use-the-oauth-2.0-state-parameter.md](../../authentication-and-access/authentication/how-to-use-the-oauth-2.0-state-parameter.md "mention")
* After login, the user will be redirected to `[redirect_uri]?code=[AUTH_CODE]`, the `AUTH_CODE` is needed in the next step
* See the supported scopes at [supported-scopes.md](../../api-reference/apis/oauth-2.0-and-openid-connect-oidc/supported-scopes.md "mention")

### Handling Callback

After receiving the `AUTH_CODE` in your application, make a POST request to the `/token` endpoint.

{% tabs %}
{% tab title="curl" %}
```sh
curl -X POST 'https://AUTHGEAR_ENDPOINT/oauth2/token' \
-H 'Content-Type: application/x-www-form-urlencoded' \
-d 'grant_type=authorization_code' \
-d 'client_id=CLIENT_ID' \
-d 'code=AUTH_CODE' \
-d 'redirect_uri=REDIRECT_URL' \
-d 'code_verifier=CODE_VERIFIER'
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const tokenEndpoint = new URL("https://AUTHGEAR_ENDPOINT/oauth2/token");
const form = new URLSearchParams();
form.set("grant_type", "authorization_code");
form.set("client_id", "CLIENT_ID");
form.set("code", AUTH_CODE); //Authorization code returned from authgear server
form.set("redirect_uri", "REDIRECT_URL");
form.set("code_verifier", code_verifier);
const response = await fetch(tokenEndpoint.toString(), {
  method: "POST",
  headers: {
    "Content-Type": "application/x-www-form-urlencoded",
  },
  body: form,
});
const responseJSON = await response.json();
```
{% endtab %}
{% endtabs %}

* Include the `code_verifier` used for generating the `code_challenge` to prove both requests come from the same client.
* The response may contain the ID token, the access token and the refresh token depending on the `scope` supplied by the previous step.

### Logout

When the user logs out, revoke the refresh token by making a POST request to the `revocation_endpoint` with the refresh token in the body in `application/x-www-form-urlencoded` as content-type. The endpoint looks like `https://AUTHGEAR_ENDPOINT/oauth2/revoke`, it can be found in OpenID Connect configuration document.

Then clear the access token and refresh token stored locally in your app.

### Verifying JWT Access Token

Include the access token in the Authorization headers of the frontend requests to verify your user’s identity. Follow this guide to learn about validating the JWT in your application server [jwt.md](../backend-api/jwt.md "mention")
