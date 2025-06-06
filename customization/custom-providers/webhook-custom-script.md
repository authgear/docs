---
description: >-
  Connect with any SMS providers with webhook or custom scripts for
  communication with the users.
---

# Webhook/Custom Script

Webhook and Custom JavaScript/TypeScript are some of the ways you can configure your Authgear project to [use Custom SMS Providers](custom-sms-provider.md).

In this guide, you'll learn how to switch from the default SMS provider to just about any third-party SMS provider.

### Prerequisites

* An SMS provider with an API endpoint that accepts external HTTPS requests.
* An Authgear project with the Custom SMS Gateway feature enabled.

{% hint style="info" %}
**Note:** Your project must be on an enterprise plan to use the Custom SMS Provider feature. [Contact us](https://www.authgear.com/schedule-demo) for more details.
{% endhint %}

### Step 1: Enable Custom SMS Gateway

To enable Custom SMS Gateway, log in to Authgear Portal, and navigate to **Advanced** > **Custom SMS Gateway**.

Toggle the **Enable Custom SMS Gateway** switch on to enable Custom SMS Provider.

<figure><img src="../../.gitbook/assets/authgear-custom-sms.png" alt=""><figcaption><p>authgear custom sms gateway</p></figcaption></figure>

### Step 2: Select SMS Gateway Provider

Select **Webhook** under SMS Gateway Provider to set up a custom SMS provider using Webhook.

Alternatively, select **Custom JS/TS** to set up a custom SMS provider using custom JavaScript/TypeScript code in your Authgear project.

### Step 3: Set up Webhook or Implement Custom Script

A key thing to consider before implementing your webhook or custom script is the structure of the payload your Authgear application will send to your endpoint.

#### Payload

The following is the structure of the JSON payload for custom SMS webhook and custom JS/TS (`CustomSMSGatewayPayload`).

```json
{
  "to": "+44123456789",
  "body": "420240 is your french verification code.\n\nDon't share it with anyone.\n",
  "app_id": "demo-enterprise",
  "template_name": "verification_sms.txt",
  "language_tag": "en",
  "template_variables": {
    "app_name": "french",
    "client_id": "tester",
    "code": "420240",
    "host": "your-app.example.com",
    "phone": "+447549293826",
    "state": "eyK1b2hjjnI6IkdaI1qwwe5KVlMySEU7Ghhjsjs"
  }
}
```

* `to` is the phone number of the receiver.
* The `body` field contains the actual message.
* `app_id`: is the identifier for the Authgear project sending the SMS.
* `template_name`: the value shows the specific sms template type that is used for composing the message `body`. For example, verification\_sms.txt when a verification SMS is sent using the verification template set in [**Email/SMS Templates**](../ui-customization/custom-email-and-sms-templates.md) in Authgear portal.
* `language_tag`: a tag specifying the language of the message.
* `template_variables`: this is an object with all the variables that are used to add dynamic content such as OTP `code`, the `app_name` of your app, etc, to the SMS template.

#### Webhook

For Webhook, follow these steps to configure the options:

* **Webhook Endpoint:** enter the full URL for the page in your application that will receive the payload from Authgear's webhook request and use it to send the SMS.
* **JSON Payload Sample:** this section shows the structure of the JSON payload. The key payload includes a `to` and a `body` field.
* **Webhook Signature Secret Key:** this field displays your project's Webhook Signature Secret Key. Each webhook event request includes a signature to confirm it comes from Authgear. We recommend that you verify this signature to confirm that the request is actually from your Authgear project before proceeding. See [Verifying Signature](../events-hooks/webhooks.md#verifying-signature).
* **Timeout:** enter a value between 1 to 60. Use a number close to the maximum if your webhook requires more time to respond.

#### Custom JS/TS

Implement the logic for sending an HTTPS request to your custom SMS provider in the **Custom script** editor.

The default sample code can be used as a good base for starting out:

```typescript
import { CustomSMSGatewayPayload, CustomSMSGatewayResponse } from "https://deno.land/x/authgear_deno_hook@v1.5.0/mod.ts";

export default async function (e: CustomSMSGatewayPayload): Promise<CustomSMSGatewayResponse> {
  const body = JSON.stringify(e);
  const response = await fetch("https://some.sms.gateway", {
    method: "POST",
    body: body,
  });

  if (!response.ok) {
    return {
      code: "delivery_rejected",
    }
  }

  return {
    code: "ok",
  }
}
```

Replace `https://some.sms.gateway` with the actual endpoint for your SMS provider.

e contains the payload (`CustomSMSGatewayPayload`) with details such as the `body` and `to` (the receiver's phone number). See the example of a [complete payload above](webhook-custom-script.md#payload).

The following example shows how to send SMS via Custom JS/TS:

```typescript
import { CustomSMSGatewayPayload, CustomSMSGatewayResponse } from "https://deno.land/x/authgear_deno_hook@v1.5.0/mod.ts";

export default async function (e: CustomSMSGatewayPayload): Promise<CustomSMSGatewayResponse> {
  const gatewayJson =  {
    "SMS": {
        "auth": {
            "username": "user@example.com",
            "apikey": "abc13de25fgh46"
        },
        "message": {
            "sender": "AG Test",
            "messagetext": e.body,
            "flash": "0"
        },
        "recipients":
        {
            "gsm": [
                {
                    "msidn": e.to
                }
            ]
        }
      }
    };
  const response = await fetch("https://api.example.com/sendsms.json", {
    method: "POST",
    body: JSON.stringify(gatewayJson),
    headers: {
      "Content-Type": "application/json"
    },
  });

  if (!response.ok) {
    return {
      code: "delivery_rejected",
    }
  }

  return {
    code: "ok",
  }
}
```

In the above example, we read the values of to and body from Authgear's `CustomSMSGatewayPayload` (e) and used it to build the format of JSON data our custom SMS gateway expects.

See the official documentation for your SMS gateway to learn more about how to structure your HTTPS request.

#### **Response**

Both the Webhook and JavaScript/TypeScript share the same response. The following shows the schema of the response your webhook or custom script should send to Authgear for better user experience and error handling:

```typescript
{
  "code":  "ok",
  "provider_error_code": ""
}
```

* `code`: this field is required, and the value can be one of the following strings:
  * `ok`: return this code if the sms is delivered successfully.
  * `invalid_phone_number`: return this code if the receiver's phone number is invalid.
  * `rate_limited`: return this code if some rate limit is reached, and the user should retry the request.
  * `authentication_failed`: return this code if some form of authentication has failed, and the developer should check the current configurations.
  * `delivery_rejected`: return this code if the sms delivery service rejected the request for any reason the user cannot fix by retrying.
* `provider_error_code`: this is an optional field to include an error code that could appear on Authgear Portal and assist debugging. A good way to use this field is to put the error code returned by your custom SMS provider (e.g., Twilio). The value should be a string.&#x20;

### Step 4: Save and Test the Custom SMS Provider

Finally, click on the **Save** button to save your new custom provider settings.

To test your work, click on the **Test** button next to the Save button, enter your phone number, and then click Send to test your new SMS provider.

