# Table of contents

* [Authgear Docs](README.md)

## Get Started

* [Choose your authentication approach](get-started/authentication-approach/README.md)
  * [Token-based (Native mobile or Single-page app)](get-started/authentication-approach/token-based.md)
  * [Cookie-based (Website or Single-page app)](get-started/authentication-approach/cookie-based.md)
* [Web SDK](get-started/website.md)
* [React Native SDK](get-started/react-native.md)
* [Flutter SDK](get-started/flutter.md)
* [Xamarin SDK](get-started/xamarin.md)
* [Android SDK](get-started/android/README.md)
  * [Android Kotlin coroutine support](get-started/android/coroutine-support.md)
  * [Android OKHttp Interceptor Extension (Optional)](get-started/android/okhttp-interceptor-extension.md)
* [iOS SDK](get-started/ios.md)
* [Backend Integration](get-started/backend-integration/README.md)
  * [Validate JWT in your application server](get-started/backend-integration/jwt.md)
  * [Forward Authentication to Authgear Resolver Endpoint](get-started/backend-integration/nginx.md)

## tutorials

* [Single-Page App](tutorials/spa/README.md)
  * [React Tutorial: User Auth with Authgear SDK](tutorials/spa/react.md)
  * [Vue Tutorial: User Auth with Authgear SDK](tutorials/spa/vue.md)
  * [Angular Tutorial: User Auth with Authgear SDK](tutorials/spa/angular.md)
* [Local Development Setup](tutorials/local-setup/README.md)
  * [Cookie-based Authorization](tutorials/local-setup/local-cookie-based-web-setup.md)

## Strategies

* [User, Identity and Authenticator](strategies/user-identity-and-authenticator.md)
* [Passkeys](strategies/passkeys.md)
* [WhatsApp OTP Login](strategies/whatsapp-otp-login.md)
* [Social & Enterprise Identity Providers](strategies/how-to-setup-sso-integrations/README.md)
  * [Connect Apps to Apple](strategies/how-to-setup-sso-integrations/apple.md)
  * [Connect Apps to Google](strategies/how-to-setup-sso-integrations/google.md)
  * [Connect Apps to Facebook](strategies/how-to-setup-sso-integrations/facebook.md)
  * [Connect Apps to GitHub](strategies/how-to-setup-sso-integrations/github.md)
  * [Connect Apps to LinkedIn](strategies/how-to-setup-sso-integrations/linkedin.md)
  * [Connect Apps to Azure Active Directory](strategies/how-to-setup-sso-integrations/azureadv2.md)
  * [Connect Apps to Microsoft AD FS](strategies/how-to-setup-sso-integrations/adfs.md)
  * [Connect Apps to Azure AD B2C](strategies/how-to-setup-sso-integrations/azureadb2c.md)
  * [Connect Apps to WeChat](strategies/how-to-setup-sso-integrations/wechat.md)
* [Biometric login](strategies/biometric.md)
* [Anonymous Users](strategies/anonymous-users.md)
* [Passwordless Login for Apple App Store Review](strategies/passwordless-demo-user-for-apple-app-review.md)
* [Ethereum & NFT](strategies/web3.md)

## Integrate

* [Using SDK to call your application server](integrate/using-sdk-to-call-your-application-server.md)
* [User Settings](integrate/auth-ui.md)
* [User Profile](integrate/user-profile.md)
* [Reauthentication](integrate/reauthentication.md)
* [How Authgear integrate with your applications](integrate/how-authgear-integrate-with-your-applications.md)
* [Single Sign-on on mobile devices](integrate/single-sign-on.md)
* [Force authentication on app launch](integrate/force-authentication-on-app-launch.md)
* [Account Deletion](integrate/account-deletion.md)
* [Using Authgear as an OpenID Connect Provider](integrate/oidc-provider.md)

## Customize

* [Privacy Policy & Terms of Service Links](customize/privacy-policy-terms-of-service.md)
* [Branding in Auth UI](customize/branding.md)
* [Custom domain](customize/custom-domain.md)
* [Custom Email Provider](customize/custom-email-provider.md)
* [Customer Support Link](customize/customer-support-link.md)
* [Localization & UI Text](customize/localization.md)

## APIs

* [API for Client Applications (OIDC 2.0)](apis/api-for-client-applications-oidc-2.0.md)
* [Admin APIs](apis/admin-apis/README.md)
  * [Generate the node id](apis/admin-apis/node-id.md)
  * [Update user's standard attributes](apis/admin-apis/update-users-standard-attributes.md)
  * [Update user's picture](apis/admin-apis/update-users-picture.md)
  * [Search for users](apis/admin-apis/search-for-users.md)

## Webhooks

* [Webhooks](webhooks/webhooks.md)

## Client App SDKs

* [Javascript SDK Reference](https://authgear.github.io/authgear-sdk-js/docs/)
* [iOS SDK Reference](https://authgear.github.io/authgear-sdk-ios/)
* [Android SDK Reference](https://authgear.github.io/authgear-sdk-android/)
* [Flutter SDK Reference](https://authgear.github.io/authgear-sdk-flutter/)
* [Xamarin SDK Reference](https://authgear.github.io/authgear-sdk-xamarin/)

## Deploy on your Cloud

* [Running locally with Docker](deploy-on-your-cloud/local.md)
* [Deploy with Helm chart](deploy-on-your-cloud/helm.md)
* [Authenticating HTTP request with Nginx](deploy-on-your-cloud/auth-nginx.md)
* [Configurations](deploy-on-your-cloud/configurations/README.md)
  * [Environment Variables](deploy-on-your-cloud/configurations/env.md)
  * [authgear.yaml](deploy-on-your-cloud/configurations/authgear.yaml.md)
  * [authgear.secrets.yaml](deploy-on-your-cloud/configurations/authgear.secrets.yaml.md)

## Security Concerns

* [Non-HTTP scheme redirect URI](security-concerns/redirect-uri.md)
