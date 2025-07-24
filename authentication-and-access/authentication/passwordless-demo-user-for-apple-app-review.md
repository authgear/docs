---
description: >-
  How to pass the Apple Store review process if your app uses passwordless
  login.
---

# Tips for Apple App Store Review with Passwordless Login

When you try to publish a mobile app on the Apple AppStore, there will be an [App Review process](https://developer.apple.com/app-store/review/). You need provide a demo user account for the reviewers to access the features of the app.

However, passwordless login via email/phone OTP cannot be used in the review because the reviewer do not have access to the email inbox or phone number of that demo account.

To work around this, you can enable "test mode" for a specific phone number or email address that allows the account to authenticate with a fixed OTP (for ex. `000000`).

## How to enable "test mode"
1. Navigate to Advanced > Edit Config
2. paste the following to the yaml: 
  ```YAML
  test_mode:
    oob_otp:
      enabled: true
      rules:
      - fixed_code: "000000"
        regex: \+85291231234$
    sms:
      enabled: true
      rules:
      - suppressed: true
        regex: \+85291231234$
    whatsapp:
      enabled: true
      rules:
      - suppressed: true
        regex: \+85291231234$
    email: 
      enabled: true
      rules:
      - suppressed: true
      regex: \me@example.com
  ```
  In this example,
  - `+85291231234` and `me@example.com` will not recieve any messages.
  - They can log in using the fixed OTP `000000`.
  - All other users will continue to receive OTPs normally.