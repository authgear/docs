---
description: >-
  How to pass the Apple Store review process if your app uses passwordless
  login.
---

# Passwordless Login for Apple App Store Review

When you try to publish a mobile app on the Apple AppStore, there will be an [App Review process](https://developer.apple.com/app-store/review/). You need provide a demo user account for the reviewers to access the features of the app.

However passwordless login via email/phone OTP cannot be used in the review because the reviewer do not have access to the email inbox or phone number of that demo account.

You can create a demo account with email/phone and password by turning password on temporarily. In the project portal:

1. Go to **Authentication > Login Methods**.
2. In **Select Login Methods**, select **Custom**.
3. In the tabs section below, select the tab **Custom Login Methods**.
4. In **Custom Login Methods**, activate **Password**.
5. Go to **User Management**, press **Add User** in the command bar.
6. **Create the demo user** by entering the email address and password
7. Go to where you were in Step 4, deactivate **Password**.
8. Now you can login as the demo user in your app with the email and password.
9. Submit your app for review with the credentials.
