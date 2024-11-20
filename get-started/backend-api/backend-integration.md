---
description: >-
  Decide how your backend application server authenticate the incoming HTTP
  requests.
---

# Backend Integration

For Mobile App or Single Page Web App or Website, each request from the client to your application server should contain an access token or a cookie. Your backend server should validate them for each HTTP request.

There are different approaches to verify the requests based on whether you validate JWT (JSON Web Tokens) in your server, or forward authentication to Authgear Resolver Endpoint.

### Validate JWT in your server

Authgear uses [JSON Web Token (JWT)](https://jwt.io/?\_gl=1\*1ybgym6\*rollup\_ga\*MTI1NDM1NjUwMy4xNjg3NzEyNTIz\*rollup\_ga\_F1G3E656YZ\*MTY5MTEzNjEzNS45NS4xLjE2OTExMzYxNDguNDcuMC4w\*\_ga\*MTI1NDM1NjUwMy4xNjg3NzEyNTIz\*\_ga\_QKMSDV5369\*MTY5MTEzNjEzNS44Ny4xLjE2OTExMzYxNDguNDcuMC4w&\_ga=2.165043391.1472871049.1691063710-1254356503.1687712523) for secure data transmission, authentication, and authorization. Tokens should be parsed and validated in regular web, native, and single-page applications to make sure the token isn’t compromised and the signature is authentic.&#x20;

Read more on [Validate JWT in your application server](jwt.md) guide.

#### JWT Token in Authorization Header

This approach is only available for [Token-based authentication](../authentication-approach/token-based.md) and involves passing the JWT token within the HTTP Authorization header. This approach is widely used in OAuth 2.0 and OIDC implementations, providing a standardized way to authenticate users.

#### JWT Token in Cookies

JWT tokens can be stored in **HTTP cookies** and sent with each request. It is suitable for  [Cookie-based authentication](../authentication-approach/cookie-based.md)**.** Storing JWTs in cookies as a way to persist the user's session across requests. The server then uses JWKS to validate the token. This approach is useful in scenarios where you want to maintain user sessions across different services in a more traditional web application setup.

{% hint style="info" %}
For Cookie-based authentication, JWT in cookies is not supported yet. [You can track the issue here](https://github.com/authgear/authgear-server/issues/1180).
{% endhint %}

### Forward Authentication to Authgear Resolver Endpoint

Forward Authentication is a process where an intermediate **reverse** **proxy or API Gateway** is responsible for authenticating a request before it reaches the intended application or service. This can add an extra layer of security and centralize the authentication logic. An intermediate service forwards each incoming HTTP request to the Authgear Resolver Endpoint to verify the access token or cookie in the HTTP header.&#x20;

Read more on [Forward Authentication to Authgear Resolver Endpoint](nginx.md) guide.

#### Forward Authorization Header

Before processing the request, your server or a reverse proxy forwards the request to an [Authgear Resolver Endpoint](nginx.md#authgear-resolver-endpoint). This endpoint resolves and verifies the authentication information (such as an Access Token) from the request **Authorization Header**.

#### Forward Cookie in HTTP header

In this pattern, Access Token (JWT) is stored in a cookie, and your server or a reverse proxy may contact the [Authgear Resolver Endpoint](nginx.md#authgear-resolver-endpoint) to obtain more information or validate certain aspects of the request.

## Comparison

|                          | **Validate JSON Web Token (JWT) in your application server**                                                                                                               | Forward Authentication to Authgear Resolver Endpoint                                                      |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Reliability              | <p><strong>Medium</strong><br>JWT only updates when expire. That means before the token expiry, your application may see the user is valid even they has been disabled</p> | <p><strong>High</strong><br>Update near real-time, based on your reserve proxy cache setting</p>          |
| Integration difficulties | <p><strong>Easy</strong><br>You only need to add code in your application to validate and decode JWT</p>                                                                   | <p><strong>Medium</strong><br>Need to setup extra reverse proxy to resolve authentication information</p> |

## Setup guides

**Validate JSON Web Token (JWT) in your application server**

{% content-ref url="jwt.md" %}
[jwt.md](jwt.md)
{% endcontent-ref %}

**Forward authentication with Authgear Resolver Endpoint**

{% content-ref url="nginx.md" %}
[nginx.md](nginx.md)
{% endcontent-ref %}
