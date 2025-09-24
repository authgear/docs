---
description: Modify the template for emails and SMS messages for your Authgear project.
---

# Custom Email and SMS Templates

Authgear allows you to modify the default Email/SMS template content for sending password reset and verification messages.

To set a custom email or SMS template, log in to Authgear Portal then navigate to **Branding** > **Email/SMS Templates**.

<figure><img src="../../.gitbook/assets/authgear-custom-email-template.png" alt=""><figcaption></figcaption></figure>

You can edit the Email/SMS Templates for each language you have enabled for your application.

**Note:** You must upgrade your [project plan](https://docs.authgear.com/reference/billing-faq) to a paid plan to edit the templates.

### Important notice for sending SMS to Chinese phone numbers (+86):

The template for SMS to Chinese numbers cannot be modified. As a result, Authgear will always use the default template for Chinese numbers even when you have set a custom template. All SMS other than OTPs (e.g. password reset links) will not be delivered.

Authgear will use your custom SMS template for other users with non-Chinese phone numbers.

The following is the content of the SMS template **regardless of locale** when sending to a Chinese phone number:&#x20;

```
【简信】你的 {app-name} 登錄驗證碼是 {code}
```

The reason for this fallback is that services are required to register their SMS templates in advance with the officials before they can deliver to Chinese numbers. Authgear has registered the default SMS template such that it will deliver without any further action on your side.

Enterprise users who still wish to customize their SMS template for Chinese numbers can [contact us](https://www.authgear.com/schedule-demo) for more details.

### Important notice for sending SMS to Vietnamese (+84) and Taiwanese (+886) phone numbers&#x20;

Due to the regulatory requirements from the local governments, all SMS messages to Vietnam & Taiwan must include the application name in the message body. If your users in Vietnam & Taiwan experience difficulties receiving SMS, try adding `[Authgear]` at the begining of the SMS template. SMS without `[Authgear]` may be blocked by the mobile operators.  For example, your OTP message could appears like:

```
[Authgear] {code} is your {app-name} one-time password.
Don't share it with anyone.
```

```
[Authgear] {code} là mã xác minh {app-name} của bạn.
Vui lòng không chia sẻ mã này với bất kỳ ai.
```

```
[Authgear] {code} 是你的 {app-name} 驗證碼。
切勿分享給任何人。
```

Enterprise users who still wish to customize the beginning part of the message for Vietnamese & Taiwanese numbers can [contact us](https://www.authgear.com/schedule-demo) for more details.
