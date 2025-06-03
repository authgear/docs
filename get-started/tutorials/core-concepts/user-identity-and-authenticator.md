# User, Identity and Authenticator

To fully configure and use Authgear, you need to understand 3 important concepts in Authgear's flexible yet powerful authentication system:

* User
* Identity
* Authenticator

Combining identity and authenticators, Authgear can support a lot of different authentication needs such as, but not included to:

* Simple email / username / phone sign in, with or without 2FA
* Email or SMS Passwordless login
* SSO login such as Google / Facebook etc
* Phone sign in with Password as the 2nd factor like telegram or whatsapp
* A lot more.

## User

A user represents an entity who use your app, usually a person. Each user can have multiple identities.

## Identity

Identities are the way users can identify themselves, such as username, email and phone. Identities could also came from an OAuth Provider such as Google/Facebook SSO, or Sign in with Apple.

Currently, Authgear support the following identity:

* Email
* Phone
* Username
* OAuth Provider:
  * Google
  * Facebook
  * Azure Active Directory
  * Linkedin
  * Sign in with Apple
* Anonymous

Each identities have its configuration. For example email could be configured to block "+" or not, or if a username is case sensitive or not.

Email, Phone and Username identities automatically create the `email`, `phone_number` and `preferred_username` claims. OAuth Provider might create a `email` claim depends on if the user's email is provided.

## Authenticator

Authenticators are the way a user with an identity can authenticate themselves, each user can multiple primary and secondary authenticators. If secondary are configured or required, a user need to authenticate itself against both to be signed in.

Currently Authgear support the following authenticators:

* Primary Authenticators:
  * Password
  * One Time Passcode via Email or SMS (oob\_otp)
* Secondary Authenticators:
  * Password
  * TOTP (2FA token)
  * One Time Passcode via Email or SMS (oob\_otp)

Secondary authentication could be optional (only if the user configured it) or required for all users.
