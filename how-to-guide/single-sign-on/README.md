---
description: >-
  Provide a seamless user experience across multiple applications with the
  single sign-on feature.
---

# Single Sign-on

Single sign-on (SSO) is defined as login once, logged in all apps. If you have multiple mobile apps or websites that wants to streamline the user experiences. You can configure your apps to turn on the SSO feature, so the end-users only have to enter their authentication credentials once.

There are multiple ways to achieve Single Sign-on, with various pros-and-cons:

|                                                 | Related Feature                                                                                                                           | Technical Remarks                                                                                             |
| ----------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| SSO between Websites with the same apex domain  | [Cookie-based Deployment](../../get-started/backend-api/#forward-cookie-in-http-header)                                                   | Requires all of the websites with the same "root domain" (e.g. app1.**example.com** and app2.**example.com**) |
| SSO between Mobile Apps and Browsers            | [SSO with Mobile App / Web SPA](sso-with-mobile-app-web-spa.md)                                                                           | Requires the use of `ASWebAuthentication` and `Custom Tab` on iOS/Android respectively                        |
| SSO between two independent mobile apps         | [App2App Login](app2app-authorization.md)                                                                                                 | Based on OIDC App2App                                                                                         |
| SSO from a Mobile App to Website                | [Pre-Authenticated URLs](pre-authenticated-urls.md)                                                                                       | Open a URL from Mobile App and pass the user session along, based on OIDC Token Exchange                      |
| SSO between Mobile Apps from the same publisher | <p>Keychain Sharing / Android Account Manager<br><br>(<a href="https://www.authgear.com/schedule-demo">Contact us</a> if you need it)</p> | Requires both apps published by the same publisher from App Store.                                            |
