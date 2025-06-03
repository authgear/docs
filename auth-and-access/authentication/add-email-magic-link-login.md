---
description: Passwordless login with email links
---

# Add Email Magic Link Login

Email Login Links, also known as "**magic link**", is a passwordless authentication method that allows users to log into a website or application without using a traditional password. Instead, it relies on a unique link sent to the user's email address.

Here's how it works:

1. User initiates the login process by entering their email address on the login page.
2. Authgear generates a unique, time-limited login link associated with the user's email address.
3. The link is sent to the user's email inbox, with a button prompting them to click on it to log in.
4. The user clicks on the link, and approve the login
5. The user is securely logged in to the app or website.

Magic link login offers several advantages. It eliminates the need for users to remember and manage passwords, reducing the risk of weak or reused passwords. It also simplifies the login process and reduces friction, as users only need to access their email to authenticate.

To enable Email Login Links:

1. In the Authgear Portal, go to "**Authentication**" > "**Login Methods**"
2. Select "**Email**" or "**Mobile/Email**" as login methods
3. Go to the "**Verification and OTP**" tab
4. Under "**Email**", in the "**Verify email by**" field, select "**Login Link**"

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>
