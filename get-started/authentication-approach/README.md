---
description: >-
  Decide how should the application requests be identified, either by access
  tokens or by cookies.
---

# Choose your authentication approach

Authgear provides token-based or cookie-based authentication. You will need to decide which approach you are going to use before starting the setup.

|                      | **Token-based**                                     | Cookie-based                                                                  |
| -------------------- | --------------------------------------------------- | ----------------------------------------------------------------------------- |
| Suitable for         | **mobile apps** or **single-page web applications** | **Websites** in the same root domain (e.g. Server-side rendered applications) |
| Transport of session | **Access Token** in `Authorization` header          | **Session ID** in Cookies                                                     |

## Token-based authentication

This approach is suitable for **mobile apps** or **single-page web applications**.

In Token-based authentication, Authgear returns the `access token` and `refresh token` to the client app after authentication.

The client SDK will automatically renew the `access token` with the `refresh token` for you, so you don't have to worry about it.

Your client app should call your backend with the access token in the Authorization header, and you can verify the access token by integrating Authgear with your backend. The HTTP requests can be authenticated by [**Forwarding to Authgear Resolver Endpoint**](../backend-integration/nginx.md) or [**Validating JWT in your application server**](../backend-integration/jwt.md).

Request example:

```bash
> GET /api_path HTTP/1.1
> Host: yourdomain.com
> Authorization: Bearer <AUTHGEAR_ACCESS_TOKEN>
```

{% content-ref url="token-based.md" %}
[token-based.md](token-based.md)
{% endcontent-ref %}

## Cookie-based authentication

This approach is suitable for **all types of websites**, including server-side rendered applications.

In Cookie-based authentication, Authgear returns `Set-Cookie` headers and sets cookies to the browser. The cookies are HTTP only and share under the same root domains. So you will need to setup the **custom domain** for Authgear, such as `identity.yourdomain.com`.

In this setting, if you have multiple applications under `yourdomain.com`, all applications would share the same session cookie automatically. After that, you can verify the cookies by integrating Authgear with your backend. The HTTP requests _must_ be authenticated by [**Forwarding to Authgear Resolver Endpoint**](../backend-integration/nginx.md).

Request example:

```javascript
> GET /api_path HTTP/1.1
> Host: yourdomain.com
> cookie: session=<AUTHGEAR_SESSION_ID>
```

{% content-ref url="cookie-based.md" %}
[cookie-based.md](cookie-based.md)
{% endcontent-ref %}

