---
description: >-
  How to pass the Apple Store review process if your app uses passwordless
  login.
---

# Tips for Apple App Store Review with Passwordless Login

When you try to publish a mobile app on the Apple AppStore, there will be an [App Review process](https://developer.apple.com/app-store/review/). You need provide a demo user account for the reviewers to access the features of the app.

However passwordless login via email/phone OTP cannot be used in the review because the reviewer do not have access to the email inbox or phone number of that demo account.

You can enable "test mode" on a specific phone number or an email address so the demo account can authenticate with a fixed OTP `000000`.

Please contact us at [hello@authgear.com](mailto:hello@authgear.com) if you need to enable test mode for a phone number or email address in your project.
