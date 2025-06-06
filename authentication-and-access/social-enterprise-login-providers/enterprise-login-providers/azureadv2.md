# Connect Apps to Azure Active Directory

## Prerequisite

1. Create an Azure Active Directory (Azure AD) account [here](https://azure.microsoft.com/free)
2. Setup a tenant by completing [Quickstart: Set up a tenant](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-create-new-tenant)
3. Register an application by completing [Quickstart: Register an application with the Microsoft identity platform](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app)
4.  Choose "Supported account type", the following options are supported:

    * Accounts in this organizational directory only (Contoso AD (dev) only - Single tenant)
    * Accounts in this organizational directory (Any Azure AD directory - Multitenant)
    * Accounts in this organizational directory (Any Azure AD directory - Multitenant) and personal Microsoft accounts (e.g. Skype, Xbox)

    "Personal Microsoft accounts only" is not supported yet. Remember the account type chosen as this affects the configuration on Authgear portal
5. Configure "Redirect URI" with `https://<YOUR_AUTHGEAR_ENDPOINT>/sso/oauth2/callback/azureadv2`
6. Follow [this](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app#add-a-client-secret) section to add a client secret. Remember to record the secret value when you add the client secret, as it will not be displayed again. This will be needed for configure OAuth client in Authgear.

{% hint style="info" %}
Redirect URI has the form of `/sso/oauth2/callback/:alias`. The `alias` is used as the identifier of OAuth provider. You can configure the `alias` in Authgear Portal.
{% endhint %}

## Configure Sign in with Microsoft through the portal

1. In the portal, go to **Authentication > Social / Enterprise Login**.
2. Enable **Sign in with Microsoft**
3. Fill in **Client ID** with **Application (client) ID** of your just created Azure AD application.
4. Fill in **Client Secret**" with the secret you get after creating a client secret for your Azure AD application.
5. For **Tenant** field:
   * If **single tenant (first option)** is chosen, fill in the **Directory (tenant) ID** of your Azure AD application.
   * If **multi tenant (second option)** is chosen, fill in the string literal `organizations`.
   * If **multi tenant and personal account (third option)** is chosen, fill in the string literal `common`.
6. **Save** the settings.

🎉 Done! You have just added Azure Active Directory (Azure AD) Login to your apps!

### Force Login page

Azure Active Directory automatically logs in to the same account without requiring a username and password. To prevent this behaviour, you can use the `prompt=login` parameter to force Azure Active Directory to show the login page. See our [guide on using the prompt=login parameter](../force-social-enterprise-login-providers-to-show-login-screen.md) in Authgear SDKs to learn more.
