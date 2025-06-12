---
description: Choose the integration approach based on application type
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: false
  pagination:
    visible: false
---

# Start Building

[![LLM | View as markdown](https://img.shields.io/badge/LLM-View%20as%20markdown-blue)](https://raw.githubusercontent.com/authgear/docs/refs/heads/main/get-started/start-building.md)

## Integration Approaches

There are 3 different high-level approaches to integrating Authgear with your applications:

1. **Mobile apps or single-page web applications:**\
   The frontend clients integrate with Authgear's SDKs, which handle full login flow and session management. It's important to validate the session in your backend server.
2. **Regular Web Applications:**\
   Traditional server-side rendered web apps that run on the server can use OIDC protocol to authenticate with Authgear. The application server has full control over the session storage.
3. **Software built by others:**\
   Integrate with other OIDC/SAML compatible applications like WordPress, Salesforce for Single Sign-On.

## **Mobile apps or single-page web applications**

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### Client-side SDKs

Client-side SDKs are designed for developers to quickly implement authentication with Auth UI on your web and mobile applications. After login, it returns the user data for your apps. It can open a hosted [pre-built account settings page](../customization/ui-customization/built-in-ui/auth-ui.md) for the user to manage their own account. The SDKs manage session token storage automatically and have built-in token ownership protection ([DPoP](https://oauth.net/2/dpop/)) against stolen refresh tokens.

**Check out the following guides for your specific framework:**

* Guides for Frontend JS SDK
  * [React](single-page-app/react.md)
  * [Vue](single-page-app/vue.md)
  * [Angular](single-page-app/angular.md)
  * [Vanilla JS](single-page-app/website.md)
* Guides for Mobile SDKs
  * [iOS](native-mobile-app/ios.md)
  * [Android](native-mobile-app/android/)
  * [React Native](native-mobile-app/react-native.md)
  * [Flutter](native-mobile-app/flutter.md)
  * [Capacitor (Ionic)](native-mobile-app/ionic-sdk.md)
  * [Xamarin](native-mobile-app/xamarin.md)
  * [Others](native-mobile-app/using-authgear-without-sdk-client-side.md)

### Validate JWT in your backend server

After the frontend integration is complete, every request sent from your application to the backend server should include the Authgear session in its header. JWKS should be used to validate the requests and decode user information from the JWT access token. See [Validate JWT in your application server](backend-api/jwt.md) for details and code examples.

### Customization

You can [customize the look and feel of Authgear prebuilt UI](../customization/ui-customization/built-in-ui/branding.md) to match your branding. [Events and hooks](../customization/events-hooks/) can be used to stay notified and add functionality during the authentication process.

### User Management through backend server

The Authgear Admin API enables comprehensive user management via a GraphQL endpoint for your backend server. The server can perform operations including searching for users, updating user details, deleting user accounts, and disabling user access.

For detailed implementation instructions and API capabilities, refer to the [Admin API ](../reference/apis/admin-api/)guide.

### Custom UI

If you wish to use a custom UI instead of the pre-built UI for signup and login, you need to deploy another server and complete the signup/login process using Authentication Flow API. See [Custom UI ](../customization/ui-customization/custom-ui/)for in-depth instructions.

## Regular Web Applications

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If your application is a traditional web app running on a server, you can leverage the OpenID Connect (OIDC) protocol to authenticate users via Authgear. A wide range of plug-and-play libraries can be found that simplify the integration process. These libraries handle crucial tasks such as authentication requests, session management, and redirecting users back to your application seamlessly.

**See the following tutorials for your specific application framework:**

* [Next JS](regular-web-app/next.js.md)
* [Express JS](regular-web-app/express.md)
* [PHP (Laravel)](regular-web-app/laravel.md)
* [Python (Flask)](regular-web-app/python-flask-app.md)
* [ASP.NET Core MVC](regular-web-app/asp.net-core-mvc.md)
* [Java (Spring Boot)](regular-web-app/java-spring-boot.md)

### Customization

You can [customize the look and feel of Authgear prebuilt UI](../customization/ui-customization/built-in-ui/branding.md) to match your branding. [Events and hooks](../customization/events-hooks/) can be used to stay notified and add functionality during the authentication process.

### User Management

The Authgear Admin API enables comprehensive user management via a GraphQL endpoint for your server. The server can perform operations including searching for users, updating user details, deleting user accounts, and disabling user access.

For detailed implementation instructions and API capabilities, refer to the [Admin API ](../reference/apis/admin-api/)guide.

### Custom UI

If you wish to use a custom UI instead of the pre-built UI for signup and login, you need to deploy another server and complete the signup/login process using Authentication Flow API. See [Custom UI ](../customization/ui-customization/custom-ui/)for in-depth instructions.

## Software built by others

When implementing identity management for your enterprise software, Authgear provides robust single sign-on (SSO) capabilities that seamlessly connect your workforce. Enterprise applications typically support standard authentication protocols like OpenID Connect (OIDC) and Security Assertion Markup Language (SAML)

* [Integration with OIDC Protocol](../how-to-guide/single-sign-on/oidc-provider.md)
* [Integration with SAML 2.0 Protocol](../how-to-guide/single-sign-on/single-sign-on-with-saml/)
