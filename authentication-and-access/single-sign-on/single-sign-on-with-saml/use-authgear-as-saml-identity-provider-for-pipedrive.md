---
description: Guide on how to use Authgear as a SAML IdP for Pipedrive
---

# Use Authgear as SAML Identity Provider for Pipedrive

Security Assertion Markup Language (SAML) is a standard that allows an Identity Provider (IdP) and a Service Provider (SP) to perform user authentication and authorization without exchanging a user's password.

In this post, you'll learn how to set up Authgear as an Identity Provider and Pipedrive as a Service Provider.

### Prerequisites

* An Authgear account. Sign up for free.
* A Pipedrive account with admin access. SAML single sign-on is available on the Pipedrive Lite plan.

### Step 1: Get the SAML Settings from Pipedrive

Log in to Pipedrive and go to **Settings** > **Security center** > **Single sign-on**.

The **SAML configuration for your Identity Provider (IDP)** section lists the values Authgear needs. Note the following:

* **Metadata url**, for example `https://<your-company>.pipedrive.com/sso/auth/samlp/metadata.xml`
* **Single Sign On (SSO) url**, for example `https://<your-company>.pipedrive.com/sso/auth/samlp`

Keep this page open. You'll come back to it in Step 5.

### Step 2: Create an Authgear Client Application

You need an Authgear client application of type `OIDC/SAML Client Application` to use Authgear as a SAML identity provider.

To create a new client application, log in to Authgear Portal, select your project then click on the **Applications** link from the navigation menu.

Next, click on **Add Application** to create a new client application. Alternatively, select an existing application of type `OIDC/SAML Client Application` and skip to step 3.

On the New Application page, enter **Name** and select `OIDC/SAML Client Application` as the **Application Type**.

Click **Save** to proceed.

### Step 3: Enable SAML 2.0 in Client Application

By default, SAML 2.0 is not enabled for the client application.

To enable SAML for your client application, click on the **SAML 2.0** tab then toggle the **SAML 2.0 Support** switch on.

Then configure the following:

* **NameID Format**: `urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified`
* **User attribute for NameID**: **Email**
* **Allowed Assertion Consumer Service URLs (ACS URLs)**: the Pipedrive **Single Sign On (SSO) url** from Step 1, for example `https://<your-company>.pipedrive.com/sso/auth/samlp`
* **Assertion Audience**: the Pipedrive **Metadata url** from Step 1, for example `https://<your-company>.pipedrive.com/sso/auth/samlp/metadata.xml`

Leave **Response Destination** and **Subject Recipient** empty.

{% hint style="info" %}
Pipedrive shows `pipedrive` as its **Issuer**, but the **Assertion Audience** field in Authgear only accepts a URL. Use the Pipedrive **Metadata url** instead.
{% endhint %}

Click on **Save** to keep your changes.

### Step 4: Send the Email Attribute

Pipedrive requires an `Email` attribute in the SAML assertion. Add it with [SAML Attribute Mapping](saml-attribute-mapping.md).

In Authgear Portal, go to **Advanced** > **Edit Config**. Under `saml.service_providers`, find the entry whose `client_id` matches your client application, and add the following `attributes` block:

```yaml
saml:
  service_providers:
  - client_id: YOUR_CLIENT_ID
    # ...existing fields, leave as-is...
    attributes:
      definitions:
      - name: Email
      mappings:
      - from:
          user_profile:
            pointer: /email
        to:
          saml_attribute: Email
```

Save the config.

### Step 5: Add Authgear SAML IdP in Pipedrive

Go back to your client application's **SAML 2.0** tab in Authgear Portal. Scroll down to the **Configuration Parameters** section, and download the **Identity Provider Certificate**.

In the Pipedrive **Single sign-on** page, fill in the **SAML configuration for Pipedrive** section:

* **Issuer**: the **Identity Provider Metadata** URL from Authgear, for example `https://<your-project>.authgear.cloud/saml2/metadata/<client-id>`
* **Single Sign On (SSO) url**: the **Login URL** from Authgear, for example `https://<your-project>.authgear.cloud/saml2/login/<client-id>`
* **X.509 certificate**: open the Identity Provider Certificate you downloaded, and paste its full content, including the `-----BEGIN CERTIFICATE-----` and `-----END CERTIFICATE-----` lines.

{% hint style="info" %}
Pipedrive only accepts a URL in the **Issuer** field, so use the **Identity Provider Metadata** URL, not the `urn:` value shown as **Issuer** in Authgear.
{% endhint %}

Click **Save changes**.

### Step 6: Test and Enable Single Sign-on

Click **Test** on the Pipedrive **Single sign-on** page. You should be redirected to the Authgear login page. Log in with an Authgear account whose email address matches a user in your Pipedrive company.

Once the test succeeds, turn on **Enable for users** and click **Save changes**. Your Pipedrive users can now log in using Authgear.
