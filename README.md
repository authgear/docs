---
description: >-
  Authgear is a highly adaptable identity-as-a-service (IDaaS) platform for web
  and mobile applications
---

# Authgear Overview

[![LLM | View as markdown](https://img.shields.io/badge/LLM-View%20as%20markdown-blue)](https://r.jina.ai/https://docs.authgear.com/)

**Authgear** is an authentication & user management solution which makes it very easy for developers to integrate and customize their consumer applications, it includes these features out of the box:

* Zero trust authentication architecture with [OpenID Connect](https://openid.net/developers/how-connect-works/) (OIDC) standard.
* Easy-to-use interfaces for user registration and login, including email, phone, username as login ID, and password, OTP, magic links, etc for authentication.
* Support a wide range of identity providers, such as [Google](https://developers.google.com/identity), [Apple](https://support.apple.com/en-gb/guide/deployment/depa64848f3a/web), and [Azure Active Directory](https://azure.microsoft.com/en-gb/products/active-directory/) (AD).
* Support biometric login on mobile, Passkeys, and Multi-Factor Authentication (MFA) such as SMS/email-based verification and authenticator apps with TOTP.
* A user management portal, like password resets, account locking, scheduled deletion or anonymization, and user profile management.
* Single Sign-On (SSO) provides a single unified experience for your customers to log into multiple web/mobile apps, including Web2Web, Web2App, and App2App SSO.
* Enable [SSO with SAML](authentication-and-access/single-sign-on/single-sign-on-with-saml/) for your users to log into multiple web applications easily.
* Session management with Authgear Portals, and a pre-built setting page for users to control concurrent sessions.
* Customizable UI with a **user-friendly low-code** dashboard.&#x20;
* Various security features such as audit logs, brute force protection, smart account lockout, password policy, etc.
* APIs for further integration and customizations. For example, build your own custom login and sign-up pages from the ground up powered by [Authentication Flow API](https://docs.authgear.com/~/changes/anTCj6yoZ06s3pLJk5v8/reference/apis/authentication-flow-api).

Most importantly, you can [get started](https://accounts.portal.authgear.com/signup) **with Authgear for free**.

### Learn about Authgear&#x20;

Authgear contains the following high-level components:

#### Authenticate on the Web/Mobile App

* **Client App SDKs** - for developers to quickly implement authentication with Auth UI on your web and mobile applications. Check out [Start Building](get-started/start-building.md) for tutorials and API References.
* **Auth UI** - is the default batteries included UI for login, signup and setting page. You can customize the style via the **Portal**, including the CSS and HTML of each pag&#x65;**.**
* [**Authentication Flow API**](customization/ui-customization/custom-ui/authentication-flow-api.md) - for developers to implement their own login, signup and reauthenticate UI (e.g. a mobile native view); or to define a customized login, signup and reauth flow.
* [**Use Authgear as OpenID Connect Provider**](authentication-and-access/single-sign-on/oidc-provider.md) - for developers to use Authgear with other software that already support OIDC login, you can use Authgear as an OpenID Connect Provider.

#### Backend Authentication and Integrations

* [**Backend/API Integration**](get-started/backend-api/) - explain the common approach of using Access Token or Cookies (JWT or random string) to authenticate an API or HTTP Requests.
* [**Admin API**](api-reference/apis/admin-api/) - allow your backend to interact directly with Authgear for user management purpose.
* [**Events and Hooks**](customization/events-hooks/) - call external web endpoint or use the hosted type-script to customize the behaviour of Authgear. E.g. blocking certain type of sign up, or call external endpoint for each login.
* [**User Import API**](admin/user-management/import-users-using-user-import-api.md) - Import multiple users from another service to your project.
* [**Export User API**](admin/user-management/export-users-using-the-user-export-api.md) - Export user data from Authgear into a CSV or [ndjosn](https://github.com/ndjson/ndjson-spec) file.
* [**Link OAuth Provider using Account Management API**](customization/ui-customization/custom-ui/manually-link-oauth-provider-using-account-management-api.md) - Link an OAuth provider to a user's account without AuthUI.

#### Management Portal

* **Authgear Portal** - You can configure your projects, manage users, check out [audit log](admin/monitor/audit-log.md), or customize the **AuthUI.** See the [5-minute quick start guide](get-started/5-minute-guide.md) for Authgear Portal.
* **Analytics Page** - View reports of all users and active users over a specific time interval on the [analytics page](admin/monitor/analytics.md).

#### Security

* [**Brute-force Protection**](security/brute-force-protection.md) - Set account Lockout Policy to safeguard a user account from brute-force login attempts.
* [**Bot Protection**](security/bot-protection.md) - Bot protection tools to block automated attackers.
* [**Password Strength**](security/password-strength.md) - Learn how to set password strength and how the password strength is calculated.

#### Login Methods

* [**Biometric Login**](authentication-and-access/authentication/biometric.md) - Add biometric login to your application.
* [**Two-Factor Authentication (2FA)**](authentication-and-access/authentication/enable-two-factor-authentication-2fa.md) - Enable 2FA in your Authgear project.
* [**Email Magic Link Login**](authentication-and-access/authentication/add-email-magic-link-login.md) - Allow users to log in without a password using a magic link.
* [**Passkeys Login**](authentication-and-access/authentication/passkeys.md) - Set up passkey for your project.
* [**Social Login / Enterprise Login**](authentication-and-access/social-enterprise-login-providers/) - Allow users to log in to your application using their existing account with a social media site or enterprise login provider.

#### Customize User Interface (UI)

* [**Customize Built-in UI**](customization/ui-customization/built-in-ui/branding.md) - Customize the look and feel of AuthUI to match your branding.
* [**Language and Localization**](customization/ui-customization/languages-and-localization.md) - Change the language for display texts.

#### User Management

Features for managing your users via Authgear Portal

* [**Create a new account on behalf of a user**](admin/user-management/how-to-handle-password-while-creating-accounts-for-users.md) - Create a new account for a user from Authgear Portal.
* [**Account Deletion**](admin/user-management/account-deletion.md) - Delete a user account from your project.
* [**User Roles and Groups**](admin/user-management/manage-users-roles-and-groups.md) - Detailed guide on how to use Roles and Groups.
* [**User Profiles**](admin/user-profiles/) - Guides on how to view and manage user profile information.
