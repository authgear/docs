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

In development and testing, you may only want certain accounts (e.g. QA team) to receive real OTPs, and have others use fixed codes.

```yaml
test_mode:
  oob_otp:
    enabled: true
    rules:
    - regex: ^\+85291231234$ # no fixed code defined for +85291231234
    - regex: .*
      fixed_code: "000000"
  sms:
    enabled: true
    rules:
    - regex: ^\+85291231234$
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

* The first rule in `oob_otp` matches `+85291231234` and has no `fixed_code` defined, so a real OTP must be used.&#x20;
* Similarly, SMS is not suppressed for `+85291231234` only, so they will receive a real OTP via SMS.
* No messages will be sent to WhatsApp or email channels

{% hint style="info" %}
Since `rules` are evaluated top to bottom, always order them from most to least specific. &#x20;
{% endhint %}

If the rules were reversed in the example above, the `.*` regex would match all accounts first, making more specific rules unreachable.\


### Case 2: Testing delivery only

Allow SMS, WhatsApp, and email messages, but always accept a specific fixed code. This is useful when you want to test message delivery, but don't want to rely on the actual code.

```yaml
test_mode:
  oob_otp:         
    enabled: true
    rules:
    - regex: .*
      fixed_code: "000000" 
```

In this example,&#x20;

* The end-to-end passwordless login flow will work as usual for all channels
* The fixed code `000000` can still be used for authentication



### Case 3: Testing specific channels

During internal testing, you may want to test only the OTP login flow for emails. To keep messaging costs low, you can suppress all SMS and WhatsApp message delivery while allowing authentication using a fixed code.&#x20;

```yaml
test_mode:
  oob_otp:         
    enabled: true
    rules:
    - regex: @   # all emails will not have fixed code
    - regex: .*
      fixed_code: "000000"
  sms:
    enabled: true
    rules:
    - regex: .*
      suppressed: true
  whatsapp:
    enabled: true
    rules:
    - regex: .*
      suppressed: true
```

In this example,

* All email accounts will receive real OTPs
* SMS and WhatsApp channels are suppressed
* The fixed code will be accepted for all non-email channels (SMS and WhatsApp)
