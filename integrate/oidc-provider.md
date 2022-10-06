---
description: Using Authgear as an OpenID Connect Provider for any OIDC compatible applications.
---

# Using Authgear as an OpenID Connect Provider for any OIDC compatible applications

If your application supports logging in using an OpenID Connect provider, you can use Authgear as the provider.

## Setting up Authgear in the Portal

1. Go to **Applications** on the left menu bar.
1. Click **⊕Add Application** in the top tool bar.
1. Input the name and select the application type **OIDC Client Application**. Click "Save".
1. You will see a link to this guide that can help you for setting up, then click "Next".
1. In the **URIs** section, fill in the **Authorized Redirect URIs** with your application's redirect uri.
1. Obtain the OpenID Connect configuration:
    1. You can obtain the **Client ID** and **Client Secret** from the **Basic Info** section.
    1. You can obtain the **OIDC Endpoints** from the **Endpoints** section.
1. Provide the OpenID Connect configuration to your application.

🎉 Done! You should be able to use Authgear to log in to your application.

## WordPress Example

In this section, we are going to demonstrate how to use Authgear as the OIDC provider for WordPress login.

1. Follow the previous section ([Setting up Authgear in the Portal](#setting-up-authgear-in-the-portal)) to setup an **OIDC Client Application**.
1. We are going to use plugin [OpenID Connect Generic Client](https://wordpress.org/plugins/daggerhart-openid-connect-generic/). Or you can use any other OIDC compatible plugin. Download and activate it in your WordPress site.
1. Go to **Setting** > **OpenID Connect Client**.
1. Fill in the form
    1. **Client ID**: Obtain the **Client ID** from the **Basic Info** section.
    1. **Client Secret Key**: Obtain the **Client Secret** from the **Basic Info** section.
    1. **OpenID Scope**: Space separated list of scopes the plugin could access.
        - Example: `openid offline_access https://authgear.com/scopes/full-userinfo`.
        - `https://authgear.com/scopes/full-userinfo` is needed to obtain user's profile (e.g. email). Otherwise the plugin will be able to get the user id only.
    1. **Login Endpoint URL**: Obtain **Authorization Endpoint** from the **Endpoints** section.
        - Example: `https://{AUTHGEAR_APP_DOMAIN}/oauth2/authorize`.
    1. **Userinfo Endpoint URL**: Obtain **Userinfo Endpoint** from the **Endpoints** section.
        - Example: `https://{AUTHGEAR_APP_DOMAIN}/oauth2/userinfo`.
    1. **Token Validation Endpoint URL**: Obtain **Token Endpoint** from the **Endpoints** section.
        - Example: `https://{AUTHGEAR_APP_DOMAIN}/oauth2/token`.
    1. **Identity Key**: Where in the user claim to find the user's identification data.
        - Suggest to use `sub` which is the user id in Authgear.
    1. **Nickname Key**: Where in the user claim array to find the user's nickname.
        - Depending on the project setup, this key must exist otherwise the login will fail. e.g. If the project is using email login, set this value to `email`.
1. At the bottom of the plugin settings page, you will be able to obtain the **Redirect URI**. Go to Authgear portal, add the uri to the **Authorized Redirect URIs**.
