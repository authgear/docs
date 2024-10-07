---
description: >-
  Authgear is a highly adaptable identity-as-a-service (IDaaS) platform for web
  and mobile applications
---

# Authgear Overview

**Authgear** is an authentication & user management solution which makes it very easy for developers to integrate and customize their consumer applications, it includes these features out of the box:

* Zero trust authentication architecture with [OpenID Connect](https://openid.net/developers/how-connect-works/) (OIDC) standard.
* Easy-to-use interfaces for user registration and login, including email, phone, username as login ID, and password, OTP, magic links, etc for authentication.
* Support a wide range of identity providers, such as [Google](https://developers.google.com/identity), [Apple](https://support.apple.com/en-gb/guide/deployment/depa64848f3a/web), and [Azure Active Directory](https://azure.microsoft.com/en-gb/products/active-directory/) (AD).
* Support biometric login on mobile, Passkeys, and Multi-Factor Authentication (MFA) such as SMS/email-based verification and authenticator apps with TOTP.
* A user management portal, like password resets, account locking, scheduled deletion or anonymization, and user profile management.
* Single Sign-On (SSO) provides a single unified experience for your customers to log into multiple web/mobile apps, including Web2Web, Web2App, and App2App SSO.
* Session management with Authgear Portals, and a pre-built setting page for users to control concurrent sessions.
* Customizable UI with a **user-friendly drag-drop low-code** dashboard.
* Various security features such as audit logs, brute force protection, smart account lockout, password policy, etc.
* APIs for further integration and customizations.

Most importantly, you can [get started](https://accounts.portal.authgear.com/signup) **with Authgear for free**.

### Learn about Authgear&#x20;

Authgear contains the following high-level components:

#### Authenticate on the Web/Mobile App

* **Client App SDKs** - for developers to quickly implement authentication with Auth UI on your web and mobile applications. Check out [Start Building](get-started/start-building/) for tutorials and API References.
* **Auth UI** - is the default batteries included UI for login, signup and setting page. You can customize the style via the **Portal**, including the CSS and HTML of each page**.**
* **Authentication Flow API** (coming soon) - for developers to implement their own login, signup and reauthenticate UI (e.g. a mobile native view); or to define a customized login, signup and reauth flow.
* [**Use Authgear as OpenID Connect Provider**](how-to-guide/authenticate/oidc-provider.md) - for developers to use Authgear with other software that already support OIDC login, you can use Authgear as an OpenID Connect Provider.

#### Backend Authentication and Integrations

* [**Backend Integration**](get-started/backend-integration/) - explain the common approach of using Access Token or Cookies (JWT or random string) to authenticate an API or HTTP Requests.
* [**Admin API**](reference/apis/admin-api/) - allow your backend to interact directly with Authgear for user management purpose.
* [**Events and Hooks**](how-to-guide/events-hooks/) - call external web endpoint or use the hosted type-script to customize the behaviour of Authgear. E.g. blocking certain type of sign up, or call external endpoint for each login

#### Management Portal

* **Authgear Portal** - You can configure your projects, manage users, check out [audit log](how-to-guide/monitor/audit-log.md), or customize the **AuthUI**
* **Analytics Page** - View reports of your total users and active users over a specific time interval on the [analytics page](how-to-guide/monitor/analytics.md).

<table data-view="cards"><thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><strong>Get Started</strong></td><td>Jump in with Authgear's getting started guides.</td><td><a href="get-started/start-building/">start-building</a></td></tr><tr><td><strong>How-To Guides</strong></td><td>Find a step-by-step guide to take your project to the next level.</td><td><a href="broken-reference">Broken link</a></td></tr><tr><td><strong>Concepts</strong></td><td>Learn the basics.</td><td><a href="broken-reference">Broken link</a></td></tr></tbody></table>
