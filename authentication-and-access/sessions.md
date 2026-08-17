---
description: >-
  Control how long users stay signed in, and whether their session survives
  after the browser is closed.
---

# Sessions

When a user signs in through Authgear — the built-in UI or your [Custom UI](../customization/ui-customization/custom-ui/README.md) — Authgear remembers the signed-in user with a session cookie on your Authgear domain (e.g. `myproject.authgear.cloud` or your custom domain). Every app in the same project shares this session — it is what powers Single Sign-On (SSO) across your apps.

Control the behavior and expiration of this session in **Authgear Portal** > **Advanced** > **Session**.

{% hint style="info" %}
Token-based apps (SPAs and native apps using the SDKs) keep their own sessions with refresh tokens. Their lifetimes are configured per application in **Portal** > **Applications**. See [Refresh Token](../reference/tokens/refresh-token.md) for details. An SDK configured with `isSSOEnabled: false` skips the shared session cookie entirely — see [SSO with mobile apps / websites](single-sign-on/sso-with-mobile-app-web-spa.md).
{% endhint %}

## Session Behavior

Session Behavior controls whether a signed-in session survives after the user closes the browser.

* **Keep users signed in** (default): The session cookie is persistent. Users stay signed in across browser restarts, until the session expires according to the [Session Expiration](sessions.md#session-expiration) settings below.
* **End session on browser close**: The session cookie is dropped when the user closes the browser, so the user must sign in again next time. Choose this when devices are shared among multiple users, such as kiosks or terminals.

{% hint style="warning" %}
**End session on browser close** relies on the browser discarding the session cookie. Browsers with session restore enabled (e.g. Chrome's "Continue where you left off") may keep session cookies across restarts. The server-side session still expires according to the Session Expiration settings.
{% endhint %}

## Session Expiration

Session Expiration controls how long a session can last before the user must sign in again. It applies regardless of the Session Behavior setting.

* **Maximum session lifetime**: The longest a session can last before it expires, even if the user stays active. The default is `31449600` seconds (52 weeks). When it expires, the user must sign in again to every app that shares the session.
* **Expire after idling** and **Idle timeout**: When enabled (default), the session expires after the user has been inactive for the idle timeout. The default is `2592000` seconds (30 days).

For example, with a maximum session lifetime of 52 weeks and an idle timeout of 30 days, a user who visits your apps at least once a month stays signed in for up to a year, while a user who is away for more than 30 days must sign in again.

## Configure with authgear.yaml

If you self-host Authgear, the same settings are available under the `session` key in `authgear.yaml`:

```yaml
session:
  # End session on browser close. Default is false (keep users signed in).
  use_session_cookie: false
  # Maximum session lifetime in seconds. Default is 31449600 (52 weeks).
  lifetime_seconds: 31449600
  # Whether the session expires after idling. Default is true.
  idle_timeout_enabled: true
  # Idle timeout in seconds. Default is 2592000 (30 days).
  idle_timeout_seconds: 2592000
```
