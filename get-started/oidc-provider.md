---
description: >-
  Using Authgear as an OpenID Connect Provider for any OIDC compatible
  applications for Single Sign-On.
---

# Integration by OIDC

If your application supports logging in using an OpenID Connect provider, you can use Authgear as the provider.

## Setting up Authgear in the Portal

1. Go to **Applications** on the left menu bar.
2. Click **⊕Add Application** in the top tool bar.
3. Input the name and select the application type **OIDC Client Application**. Click "Save".
4. You will see a link to this guide that can help you for setting up, then click "Next".
5. In the **URIs** section, fill in the **Authorized Redirect URIs** with your application's redirect uri.
6. Obtain the OpenID Connect configuration:
   1. You can obtain the **Client ID** and **Client Secret** from the **Basic Info** section.
   2. You can obtain the **OIDC Endpoints** from the **Endpoints** section.
7. Provide the OpenID Connect configuration to your application.

🎉 Done! You should be able to use Authgear to log in to your application.

## WordPress Example

In this section, we are going to demonstrate how to use Authgear as the OIDC provider for WordPress login.

1. Follow the previous section ([Setting up Authgear in the Portal](oidc-provider.md#setting-up-authgear-in-the-portal)) to setup an **OIDC Client Application**.
2. We are going to use plugin [OpenID Connect Generic Client](https://wordpress.org/plugins/daggerhart-openid-connect-generic/). Or you can use any other OIDC compatible plugin. Download and activate it in your WordPress site.
3. Go to **Setting** > **OpenID Connect Client**.
4. Fill in the form
   1. **Client ID**: Obtain the **Client ID** from the **Basic Info** section.
   2. **Client Secret Key**: Obtain the **Client Secret** from the **Basic Info** section.
   3. **OpenID Scope**: Space separated list of scopes the plugin could access.
      * Example: `openid offline_access https://authgear.com/scopes/full-userinfo`.
      * `https://authgear.com/scopes/full-userinfo` is needed to obtain user's profile (e.g. email). Otherwise the plugin will be able to get the user id only.
   4. **Login Endpoint URL**: Obtain **Authorization Endpoint** from the **Endpoints** section.
      * Example: `https://{AUTHGEAR_APP_DOMAIN}/oauth2/authorize`.
   5. **Userinfo Endpoint URL**: Obtain **Userinfo Endpoint** from the **Endpoints** section.
      * Example: `https://{AUTHGEAR_APP_DOMAIN}/oauth2/userinfo`.
   6. **Token Validation Endpoint URL**: Obtain **Token Endpoint** from the **Endpoints** section.
      * Example: `https://{AUTHGEAR_APP_DOMAIN}/oauth2/token`.
   7. **End Session Endpoint URL**: Keep it empty.
   8. **Identity Key**: Where in the user claim to find the user's identification data.
      * Suggest to use `sub` which is the user id in Authgear.
   9. Setup the user claim keys based on your project login method setting.
      * If your project is using **email** to login
        * **Nickname Key**: Set it to `email`.
        * **Email Formatting**: Set it to `{email}`.
      * If your project is using **phone** to login
        * **Nickname Key**: Set it to `phone_number`.
        * **Email Formatting**: Clear it.
      * If your project is using **username** to login
        * **Nickname Key**: Set it to `preferred_username`.
        * **Email Formatting**: Clear it.
5. At the bottom of the plugin settings page, you will be able to obtain the **Redirect URI**. Go to Authgear portal, add the uri to the **Authorized Redirect URIs**.
