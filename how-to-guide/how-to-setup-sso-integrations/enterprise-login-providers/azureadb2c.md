# Connect Apps to Azure AD B2C

## Prerequisite

1. Sign in [Microsoft Azure](https://portal.azure.com/).
2. Create a B2C tenant by following [this tutorial](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-create-tenant).
3. Enable self-service sign-up for the tenant by following [this doc](https://docs.microsoft.com/en-us/azure/active-directory/external-identities/self-service-sign-up-user-flow#enable-self-service-sign-up-for-your-tenant)
4. Go back the main page of [Microsoft Azure](https://portal.azure.com/) and search for "Azure AD B2C"
5. Create a app registration for Authgear by following [this guide](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-register-applications?tabs=app-reg-ga).
6. Configure "Redirect URI" with `https://<YOUR_AUTHGEAR_ENDPOINT>/sso/oauth2/callback/azureadb2c`.
7. Follow [this guide](https://docs.microsoft.com/en-us/azure/active-directory-b2c/add-sign-up-and-sign-in-policy?pivots=b2c-user-flow) to create a sign-up and sign-in user flow.
8. After creating the user flow, configure it

* Open "Application Claims".
* Make sure "Email Addresses" is checked.

## Configure Sign in with Azure AD B2c through the portal

If you have finished the above prerequisite, you should have the following information:

1. The **Tenant Name**, obtained in Step 2
2. The **Application (Client) ID**, obtained in Step 5
3. The **Policy (User flow) Name**, obtained in Step 7

Then in Authgear portal, do the following:

1. In the portal, go to **Authentication > Social / Enterprise Login**.
2. Enable **Sign in with Microsoft Azure AD B2C**.
3. Fill in **Client ID** with the **Application (Client) ID** above.
4. Fill in **Client secret** with the client secret you get when you create the app registration.
5. Fill in **Tenant** with the Azure AD B2C **Tenant Name**.
6. Fill in **Policy** with the **Policy (User Flow) Name**. Normally it starts with `b2c_`.
7. **Save** the changes

🎉 Done! You have just added Azure AD B2C Login to your apps!

### Force Login page

Azure AD B2C automatically logs in to the same account without requiring a username and password. To prevent this behaviour, you can use the `prompt=login` parameter to force Azure AD B2C to show the login page. See our [guide on using the prompt=login parameter](../force-social-enterprise-login-providers-to-show-login-screen.md) in Authgear SDKs to learn more.
