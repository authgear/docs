---
description: Guide on how to use Authgear as a SAML IdP for Notion
---

# Use Authgear as SAML Identity Provider for Notion

Security Assertion Markup Language (SAML) is a standard that allows an Identity Provider (IdP) and a Service Provider (SP) to perform user authentication and authorization without exchanging a user's password.

In this post, you'll learn how to set up Authgear as an Identity Provider and Notion as a Service Provider.

### Prerequisites

* An Authgear account. Sign up for free.
* Email login enabled in your Authgear project. Notion identifies users by email address, so every user who signs in to Notion needs one.
* A Notion workspace on the Business or Enterprise plan, and a workspace owner account.
* At least one email domain verified in Notion. See [Domain management](https://www.notion.com/help/domain-management) in the Notion help center.

### Step 1: Get the SAML Settings from Notion

Log in to Notion and go to **Settings** > **Identity**. In the **SAML single sign-on (SSO)** section, click **Edit SAML SSO configuration**.

Note the following:

* **Assertion Consumer Service (ACS) URL**, for example `https://app.notion.com/sso/saml/<workspace-id>`
* **SAML SSO Entity ID**, for example `https://app.notion.com/sso/saml`

Keep this dialog open. You'll come back to it in Step 6.

{% hint style="info" %}
On the Enterprise plan, SAML is set up per organization instead. Go to **Manage organization** > **General** > **SAML configurations** and click **Create configuration**.
{% endhint %}

### Step 2: Create an Authgear Client Application

You need an Authgear client application with SAML 2.0 support to use Authgear as a SAML identity provider.

1. Log in to Authgear Portal, select your project, and go to **Applications**.
2. Click **Add Application**.
3. Enter a **Name** for the application, for example `Notion`.
4. Under **Framework**, in the **Other** section, select **Other OIDC/SAML compatible**.
5. Click **Create application**.

The application opens on its **Quick Start** tab.

{% hint style="info" %}
Only applications created with **Other OIDC/SAML compatible** have a **SAML 2.0** tab. To reuse an existing application, check that it has this tab, then skip to Step 3.
{% endhint %}

### Step 3: Enable SAML 2.0 in Client Application

By default, SAML 2.0 is not enabled for the client application.

To enable SAML for your client application, click on the **SAML 2.0** tab then toggle the **SAML 2.0 Support** switch on.

Then configure the following:

* **NameID Format**: `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`
* **Allowed Assertion Consumer Service URLs (ACS URLs)**: the Notion **Assertion Consumer Service (ACS) URL** from Step 1
* **Assertion Audience**: the Notion **SAML SSO Entity ID** from Step 1
* **Also include ACS URL as Assertion Audience**: off

Leave **Response Destination** and **Subject Recipient** empty.

{% hint style="warning" %}
Notion rejects a SAML response with more than one audience. Turn off **Also include ACS URL as Assertion Audience**, or Notion fails the sign-in.
{% endhint %}

Click on **Save** to keep your changes.

### Step 4: Send the Email Attribute

Notion requires an `emailAddress` attribute in the SAML assertion. Add it with [SAML Attribute Mapping](saml-attribute-mapping.md).

In Authgear Portal, go to **Advanced** > **Edit Config**. Under `saml.service_providers`, find the entry whose `client_id` matches your client application, and add the following `attributes` block:

```yaml
saml:
  service_providers:
  - client_id: YOUR_CLIENT_ID
    # ...existing fields, leave as-is...
    attributes:
      definitions:
      - name: emailAddress
      mappings:
      - from:
          user_profile:
            pointer: /email
        to:
          saml_attribute: emailAddress
```

Save the config.

{% hint style="info" %}
Notion also reads the optional `firstName` and `lastName` attributes. Map them from `/given_name` and `/family_name` the same way only if your users have those fields set.
{% endhint %}

### Step 5: Get the Authgear SAML Settings

Go back to your client application's **SAML 2.0** tab in Authgear Portal and note the **Identity Provider Metadata** URL, for example `https://<your-project>.authgear.cloud/saml2/metadata/<client-id>`.

### Step 6: Add Authgear SAML IdP in Notion

In the Notion **SAML SSO configuration** dialog, under **Identity Provider Details**, enter the **Identity Provider Metadata** URL from Step 5 as the **IdP URL**.

Save the configuration, then turn on **Enable SAML SSO**.

{% hint style="info" %}
Keep a workspace owner signed in to Notion in another browser while you test, so you can fix the setup if the test fails.
{% endhint %}

### Step 7: Test Single Sign-on

Open a private browser window and go to [https://app.notion.com/login](https://app.notion.com/login). Enter the email address of a user in your verified domain and continue.

You should be redirected to the Authgear login page. Log in with the Authgear account that has the same email address. Notion then signs you in.

Your Notion users can now log in using Authgear.
