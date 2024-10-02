---
description: This concept document are to explain how the SDKs work.
---

# How Authgear integrate with your applications

{% hint style="info" %}
For simplicity, you should use Authgear SDKs (JS, iOS, Android) to integrate your application with Authgear.
{% endhint %}

The SDKs are actually a simple wrapper -- behind the scene, there are two recommended ways to authenticate and maintain a session when you use Authgear.

This concept document are to explain how the behind-the-scene works.

## Token-based

### Mobile or Single Page Web Applications

The JS and Mobile SDKs authenticate with Authgear at `[your app name].authgear.com/oauth2/authorize?...`, according to the OIDC 2.0 standard.

If authentication is successful, the `/oauth2/authorize` returns an `access token` and `refresh token` to the Client App.

The SDKs will automatically renew the `access token` with the `refresh token` for you, so you don't have to worry about it.

Your app should call your backend with the access token on the Authorization header, and you can verify the access token with`[your app name].authgear.com/_resolver/resolve` ([more details here](../get-started/backend-api/nginx.md)); Alternatively, the access token is a JWT signed token which you can verify locally on your backend too.

Finally, the application could call `[your app name].authgear.com/oauth2/revoke` to logout.

Hence, here are a few implications you need to understand when using this type of integration:

* Some features, such as disable or logout a session from management portal, would only be effective when the access token expire next time, and hence you may want to set a smaller value for "Access Token Lifetime".
* You may want to verify the JWT access token to avoid the latency to the `/_resolver/resolve` endpoint

## Cookie-based

### Web Applications (Server side rendered) under the same root domains

If you host a Web Application which is server side rendered, such as Ruby on Rails, PHP, ASP .NET etc, under `yourdomain.com`, you are recommended to set up a custom domain for Authgear, such as `identity.yourdomain.com`

In this setting, if you have multiple applications under `yourdomain.com`, all applications would share the same session cookies automatically.

To login, you would redirect users to `identity.yourdomain.com/oauth2/authorize?` with a `redirect_uri` parameter set. After the authentication is successful, Authgear will redirect the users to the URI, and set a cookie with a session access token.

Your backend could verify the session access token via `/_resolver/resolve`.

### Web Applications (Server side rendered) under a different domains.

If your server side rendered Web Application is hosted under`otherapp.com`, but you wish to authenticate users under `identity.yourdomain.com` instead of setting another custom domain at `identity.otherapp.com` for a better single sign on user experience, conceptually you need to do the following:

1. Authenticate users by redirecting to `identity.yourdomain.com/oauth2/authorize?...`
2. After successful, get the `access token` and `refresh token` from `identity.yourdomain.com/oauth2/token`
3. Create your own cookie session at `otherapp.com` , you can use the access token for the cookie content.
4. As soon as the access token expires, get a new access token with your refresh token.
5. Clear your cookie session if user requested, or failed to get the new access token with refresh tokens (which meant the session was revoked)

See [Using Authgear as an OIDC Provider](../how-to-guide/authenticate/oidc-provider.md) for how to set this up.

{% hint style="success" %}
TODO: Write an advanced guide for this situation.
{% endhint %}
