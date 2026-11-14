---
description: Guide on how to use Authgear as a SAML IdP for Google Workspace
---

# Use Authgear as SAML Identity Provider for Google Workspace

Security Assertion Markup Language (SAML) is a standard that allows an Identity Provider (IdP) and a Service Provider (SP) to perform user authentication and authorization without exchanging a user's password.

In this post, you'll learn how to set up Authgear as an Identity Provider and Google Workspace as a Service Provider. Your Google Workspace users will then sign in to Gmail, Drive, and other Google services through Authgear.

### Prerequisites

* An Authgear account. Sign up for free.
* Email login enabled in your Authgear project. Google Workspace identifies users by their primary email address, so every user who signs in needs an Authgear account with that same email address.
* A Google Workspace account with super admin access.

### Step 1: Create an Authgear Client Application

You need an Authgear client application with SAML 2.0 support to use Authgear as a SAML identity provider.

1. Log in to Authgear Portal, select your project, and go to **Applications**.
2. Click **Add Application**.
3. Enter a **Name** for the application, for example `Google Workspace`.
4. Under **Framework**, in the **Other** section, select **Other OIDC/SAML compatible**.
5. Click **Create application**.

The application opens on its **Quick Start** tab.

{% hint style="info" %}
Only applications created with **Other OIDC/SAML compatible** have a **SAML 2.0** tab. To reuse an existing application, check that it has this tab, then skip to Step 2.
{% endhint %}

### Step 2: Get the Authgear SAML Settings

Click on the **SAML 2.0** tab and note the following:

* **Issuer**, for example `urn:<your-project>.authgear.cloud`
* **Login URL**, for example `https://<your-project>.authgear.cloud/saml2/login/<client-id>`

In the **Identity Provider Certificates** section, click **Download Certificate**.

### Step 3: Add a SAML Profile in Google Workspace

1. Log in to the [Google Admin console](https://admin.google.com) and go to **Security** > **Authentication** > **SSO with third-party IdP**.
2. In **Third-party SSO profiles**, click **Add SAML profile**.
3. Enter a name for the profile, for example `Authgear`.
4. Fill in **IDP details**:
   * **IDP entity ID**: the **Issuer** from Step 2, for example `urn:<your-project>.authgear.cloud`
   * **Sign-in page URL**: the **Login URL** from Step 2, for example `https://<your-project>.authgear.cloud/saml2/login/<client-id>`
   * **Sign-out page URL**: leave empty
   * **Verification certificate**: upload the Identity Provider Certificate you downloaded in Step 2
5. Leave **Use a domain specific issuer** unchecked.
6. Click **Save**.

Google then shows the **SP details** of the profile. Note the following:

* **Entity ID**, for example `https://accounts.google.com/samlrp/<profile-id>`
* **ACS URL**, for example `https://accounts.google.com/samlrp/<profile-id>/acs`

{% hint style="info" %}
Google Workspace doesn't support SAML Single Logout. Signing out of Google doesn't sign the user out of Authgear, so leave **Single Logout** off in Authgear.
{% endhint %}

### Step 4: Enable SAML 2.0 in Client Application

By default, SAML 2.0 is not enabled for the client application.

Go back to your client application's **SAML 2.0** tab in Authgear Portal and toggle the **SAML 2.0 Support** switch on.

Then configure the following:

* **NameID Format**: `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`
* **Allowed Assertion Consumer Service URLs (ACS URLs)**: the Google **ACS URL** from Step 3, for example `https://accounts.google.com/samlrp/<profile-id>/acs`
* **Assertion Audience**: the Google **Entity ID** from Step 3, for example `https://accounts.google.com/samlrp/<profile-id>`

Leave **Response Destination** and **Subject Recipient** empty.

Click on **Save** to keep your changes.

### Step 5: Assign the SAML Profile

The SAML profile only takes effect for the users it's assigned to.

{% hint style="warning" %}
Start with an organizational unit or group that holds a test user. If the setup is wrong, the users you assign can't sign in to Google until you change the assignment back.
{% endhint %}

1. In the Google Admin console, go back to **Security** > **Authentication** > **SSO with third-party IdP**.
2. Click **Manage SSO profile assignments**.
3. Select the organizational unit or group to assign.
4. Under **SSO profile assignment**, select **Another SSO profile**, then select the profile you created in Step 3.
5. Click **Save**.

Google can take a few minutes to apply the change.

### Step 6: Test Single Sign-on

Open a private browser window and go to [https://accounts.google.com](https://accounts.google.com). Enter the email address of a user in the organizational unit or group you assigned.

You should be redirected to the Authgear login page. Log in with the Authgear account that has the same email address. Google then signs you in.

Once the test succeeds, assign the profile to the rest of your organization. Your Google Workspace users can now log in using Authgear.

### Troubleshooting

If Google shows an error after you log in to Authgear, check the following:

* The Authgear account's email address is the user's primary email address in Google Workspace.
* **Assertion Audience** in Authgear is exactly the **Entity ID** Google shows for the profile.
* The certificate uploaded to Google is the current Identity Provider Certificate. If you rotate the certificate in Authgear, upload the new one to Google.

To see why Google rejected a sign-in, go to **Reporting** > **Audit and investigation** > **SAML log events** in the Google Admin console.
