---
description: Configure password reset/account recovery processes.
---

# Change Forgot/Reset Password settings

The Forgot/Reset Password settings tab allows you to configure the behaviour of the account recovery process for your Authgear project.&#x20;

## Enable Reset Password

Select a login method and navigate to the password settings tab. The login method chosen controls how the user can reset their password:

* Phone (OTP only)
* Email (link or OTP)
* Both

If both options are available, the user will be able to choose between using phone or email for account recovery.&#x20;

### Recovery via phone

You can choose how you want the OTP to be delivered. In the following example, the user will receive an OTP via "WhatsApp or SMS". This means the user can select their preferred method during reset.

<figure><img src="../../../.gitbook/assets/Screenshot 2025-08-15 at 00.12.50.png" alt=""><figcaption></figcaption></figure>

The valid duration (in seconds) for the OTP or reset link can also be customized. This duration does **not** determine the minimum delay before resending OTPs or links.

As an example, if OTP is enabled as the recovery method, users will see the following screen to verify their identity during password reset:&#x20;

<figure><img src="../../../.gitbook/assets/Screenshot 2025-08-15 at 20.53.16.png" alt=""><figcaption></figcaption></figure>

Once you're done, save your changes to enable the new configuration.
