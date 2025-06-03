---
description: Add Google Sign in to your apps in less than 5 minutes.
---

# Connect Apps to Google

## Set up OAuth client on Google Cloud Platform

To configure Google OAuth client for Authgear, you will need to create an OAuth client on Google Cloud Platform first.

### Create a new project

Create a project on Google Cloud Platform through [console](https://console.cloud.google.com/). If you are adding Authgear to your existing Google Cloud Platform projects, you may skip to the next step to create the OAuth client.

### Create OAuth Consent Screen

After creating a new project, you will need to configure the OAuth consent screen. Press the <img src="../../../.gitbook/assets/Google-hamburger-menu.png" alt="" data-size="line"> button on the top-left and go to **APIs & Services** -> **OAuth consent screen** and follow the instruction to create the consent screen.

### Create OAuth client ID

1. Go to <img src="../../../.gitbook/assets/Google-hamburger-menu.png" alt="" data-size="line"> -> **APIs & services** -> **Credentials**
2. Click **Create Credentials** -> **OAuth client ID**
3. Choose **Web application** in **Application type** and assign a name as reference. You should _always choose Web application_ here regardless of the platform of the app you are creating. It is because this OAuth Client ID is used by your Authgear services, which is a web application in Google’s classification.
4. In **Authorized JavaScript origins**, add your Authgear endpoint, e.g. `https://myproject.authgear.cloud`
5. In **Authorized redirect URIs**, add `https://<YOUR_AUTHGEAR_ENDPOINT>/sso/oauth2/callback/google`. For example, `https://myproject.authgear.cloud/sso/oauth2/callback/google`
6. After creating a client ID, you will see the client ID under the **OAuth 2.0 Client IDs** section of the **Credentials** page.

![OAuth Client ID in the Credentials page](../../../.gitbook/assets/GCP_create_client_id.png)

{% hint style="info" %}
Redirect URI has the form of `/sso/oauth2/callback/:alias`. The `alias` is used as the identifier of OAuth provider. You can configure the `alias` in Authgear Portal.
{% endhint %}

You can find more details in [official Google Cloud Platform doc](https://support.google.com/cloud/answer/6158849)

## Configure Sign in with Google in Authgear Portal

### Get your OAuth Client details

After creating an OAuth client, click the name of OAuth client to view the details.

![Get your OAuth Client ID and Secret in the details page](../../../.gitbook/assets/gcp_client_details.png)

You will need the values of **Client ID**, **Client secret** to configure Google Sign In.

### Configure in Authgear Portal

1. In the portal, go to **Authentication > Social / Enterprise Login**.
2. Enable **Sign in with Google**.
3. Fill in the **Client ID** and **Client Secret** with the values obtained from the previous step.
4. **Save** the settings.

🎉Done! You have just added Google Sign In to your apps!

Your end-users can now sign in with Google on Authgear pre-built Log In and Sign Up page. Existing end-users can connect their account to Google in the [User Settings](../../../design/built-in-ui/auth-ui.md) page.

!["Sign in with Google" in Log in and Sign up page](../../../.gitbook/assets/google_sign_in.png)

![Your end-users can connect to their Google account in User Settings page](../../../.gitbook/assets/connect_with_google.png)
