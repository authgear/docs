---
description: Suppress message delivery and fix OTP for testing purposes
---

# OTP Test Mode

Test mode allows for control over how OTPs are handled during development and testing.&#x20;

Common use cases include:

* Internal testing (e.g. Allowing specific accounts to authenticate with fixed codes for ease of testing).
* Testing SMS, WhatsApp, or email login via OTP, but don't want to rely on the actual OTP delivery.
* App Store Reviewers. See [Tips for Apple App Store Review with Passwordless Login](../authentication-and-access/authentication/passwordless-demo-user-for-apple-app-review.md).

## How to enable test mode

In your portal, navigate to Advanced > Edit Config

<figure><img src="../.gitbook/assets/Screenshot 2025-07-24 at 14.34.27.png" alt=""><figcaption></figcaption></figure>



Then add the following to the yaml:

```yaml
test_mode:
  oob_otp:          # rules defined below apply to all OOB channels (email, sms, whatsapp)
    enabled: true
    rules:
    - regex: .*
      fixed_code: "000000" 
  sms:              # channel specific rules defined here
    enabled: true
    rules:
    - regex: .*
      suppressed: true
```

How this works

* `regex`: Defines target emails/phone numbers the rule applies to using a regular expression
* `fixed_code`: A 6-digit code accepted as the OTP for targets
* `suppressed`: `false` by default. If `true`, messages will not be sent to the target
* `enabled`
  * Set to `true` to enable test mode
  * `false` by default. This means the OTP flow will work as expected, and fixed codes will not be accepted for the channel.

## Examples

### Case 1: Development

In development, you may want to use a fixed code for testing.

```yaml
test_mode:
  oob_otp:
    enabled: true
    rules:
    - regex: .*
      fixed_code: "000000"
  sms:
    enabled: true
    rules:
    - regex: ^\+85291231234$ # SMS not suppressed for +85291231234
    - regex: .*
      suppressed: true
  whatsapp:
    enabled: true
    rules:
    - regex: .*
      suppressed: true
  email:
    enabled: true
    rules:
    - regex: .*
      suppressed: true
```

In this case,

* The rule in `oob_otp` matches all target so a fixed OTP `000000` will be used.&#x20;
* SMS is not suppressed for `+85291231234` only. They will receive `000000` via SMS so the SMS sending capability can be tested.
* No messages will be sent to WhatsApp or email channels

{% hint style="info" %}
Since `rules` are evaluated top to bottom, always order them from most to least specific. &#x20;
{% endhint %}

If the rules in `sms` were reversed in the example above, the `.*` regex would match all accounts first, making more specific rules unreachable.

### Case 2: App review

During app review, you can enable "test mode" for a specific phone number or email address that allows the account to authenticate with a fixed OTP. So this account can be shared with the app reviewer to complete the passwordless login flow

```yaml
test_mode:
  oob_otp:
    enabled: true
    rules:
    - regex: ^\+85291231234$
      fixed_code: "000000"
  sms:
    enabled: true
    rules:
    - regex: ^\+85291231234$
      suppressed: true
  whatsapp:
    enabled: true
    rules:
    - regex: ^\+85291231234$
      suppressed: true
```

In this example,

* Only `+85291231234` can be passed with the fixed code `000000` and other accounts should use real OTP.
* SMS and WhatsApp are suppressed for `+85291231234`
