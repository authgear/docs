# Connect Apps to Microsoft Entra ID (Azure AD)

{% hint style="info" %}
**Microsoft Entra ID** is the new name for **Azure Active Directory (Azure AD)**. Microsoft renamed the product in 2023; existing Azure AD configurations and integrations continue to work without change. This page uses the current name, but you may still see "Azure AD" in older portals or documentation.
{% endhint %}

## Prerequisite

1. Create a Microsoft Entra ID account [here](https://azure.microsoft.com/free)
2. Setup a tenant by completing [Quickstart: Set up a tenant](https://learn.microsoft.com/en-us/entra/fundamentals/create-new-tenant)
3. Register an application by completing [Quickstart: Register an application with the Microsoft identity platform](https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app)
4.  Choose "Supported account types", the following options are supported:

    * Single tenant only - &lt;your tenant&gt; (Accounts in this organizational directory only)
    * Multiple Entra ID tenants (Accounts in any organizational directory - Multitenant)
    * Any Entra ID tenant + Personal Microsoft accounts (Multitenant and personal Microsoft accounts, e.g. Skype, Xbox)

    "Personal accounts only" is not supported yet. Remember the account type chosen as this affects the configuration on Authgear portal
5. Configure "Redirect URI" with `https://<YOUR_AUTHGEAR_ENDPOINT>/sso/oauth2/callback/azureadv2`. See [How to add a redirect URI to your application](https://learn.microsoft.com/en-us/entra/identity-platform/how-to-add-redirect-uri).
6. Follow [this](https://learn.microsoft.com/en-us/entra/identity-platform/how-to-add-credentials?tabs=client-secret) section to add a client secret. Remember to record the secret value when you add the client secret, as it will not be displayed again. This will be needed for configure OAuth client in Authgear.

{% hint style="info" %}
Redirect URI has the form of `/sso/oauth2/callback/:oauth_provider_alias`. The `oauth_provider_alias` is the OAuth Provider Alias configured for this provider in Authgear Portal.
{% endhint %}

## Configure Sign in with Microsoft through the portal

1. In the portal, go to **Authentication > Social / Enterprise Login**.
2. Enable **Sign in with Microsoft**
3. Fill in **Client ID** with **Application (client) ID** of your just created Microsoft Entra ID application.
4. Fill in **Client Secret**" with the secret you get after creating a client secret for your Microsoft Entra ID application.
5. For **Tenant** field:
   * If **single tenant (first option)** is chosen, fill in the **Directory (tenant) ID** of your Microsoft Entra ID application.
   * If **multi tenant (second option)** is chosen, fill in the string literal `organizations`.
   * If **multi tenant and personal account (third option)** is chosen, fill in the string literal `common`.
6. **Save** the settings.

🎉 Done! You have just added Microsoft Entra ID (Azure AD) Login to your apps!

### Force Login page

Microsoft Entra ID automatically logs in to the same account without requiring a username and password. To prevent this behaviour, you can use the `prompt=login` parameter to force Microsoft Entra ID to show the login page. See our [guide on using the prompt=login parameter](../force-social-enterprise-login-providers-to-show-login-screen.md) in Authgear SDKs to learn more.
