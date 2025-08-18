---
description: >-
  Requiring users to reset their password if they haven't logged in after
  specific number of days
---

# Password Expiry

You can set up your Authgear project such that a user's password expires after a specific number of days. When a user logs in after the password expiry date, they'll see a prompt to change their password before they're redirected back to your app.&#x20;

{% hint style="info" %}
By default, password expiry is turned **off** for your Authgear project. [Recent security research](https://www.ncsc.gov.uk/blog-post/problems-forcing-regular-password-expiry) shows that forcing users to change their passwords after some time can do more harm than good.
{% endhint %}

## Enable Password Expiry

Navigate to the password settings tab and scroll to the **Password Expiry** section. Toggle the "**Force password change on next login if it has expired"** button to enable password expiry.

<figure><img src="../../../.gitbook/assets/Screenshot 2025-08-05 at 14.58.11.png" alt=""><figcaption></figcaption></figure>

## Set Expiry Date

You can use the field **Force change since last update (days)** to specify the number of days after which a user's password should expire.

For example, setting the value to 90 means the user's password will expire 90 days after the day they set or updated their password.

Once you're done, hit the **Save** button to keep your changes.
