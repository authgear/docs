# User, Identity and Authenticator

Authgear’s authentication system is flexible and powerful. To configure it effectively, it helps to understand three core concepts:

* User
* Identity
* Authenticator

By combining identities and authenticators, Authgear enables a wide range of authentication scenarios, such as:

* Simple email / username / phone sign-in, with or without 2FA
* Email or SMS passwordless login
* Social logins (Google, Facebook, etc.)
* Phone sign-in with password as a second factor (similar to Telegram or WhatsApp)
* And much more

## User

A **User** represents an entity that interacts with your application — typically a person.

Each user can have **multiple identities** that represent different ways of identifying themselves.

## Identity

An **Identity** is a piece of information that uniquely identifies a user.\
Examples include username, email, phone number, or identities issued by OAuth providers.

Authgear currently supports the following identity types:

* Email
* Phone
* Username
* Anonymous
* [Social/Enterprise Login](../authentication-and-access/social-enterprise-login-providers/)

Each identity type has its own configuration options. For example:

* Email identities can be configured to allow or block “+” tags.
* Username identities can be case-sensitive or case-insensitive.

When a user registers with an Email, Phone, or Username identity, Authgear automatically creates the corresponding OIDC standard claims:

* `email`
* `phone_number`
* `preferred_username`

OAuth provider identities may also provide claims such as `email`, depending on the provider and the user’s consent.

#### Identity Uniqueness

Each identity is **uniquely linked to a single user**.\
This means two users cannot share the same:

* Email address
* Phone number
* Username

## Authenticator

An **Authenticator** is a method a user uses to prove their identity. Each user may have multiple **primary** and **secondary** authenticators. If [secondary authentication](../authentication-and-access/authentication/enable-two-factor-authentication-2fa.md) is required, users must pass both primary and secondary authentication to sign in.

Authgear supports two categories of authenticators:

#### 1. Primary Authenticators

* Password
* One-Time Passcode (Email) — `oob_otp_email`
* One-Time Passcode (SMS/WhatsApp) — `oob_otp_sms`

Primary OTP authenticators are automatically created based on the user’s identities. For example:

* If the user has an email identity and OTP-via-email login is enabled, Authgear automatically creates an `oob_otp_email` authenticator.
* If the user has a phone identity and OTP-via-SMS/WhatsApp login is enabled, Authgear automatically creates an `oob_otp_sms` authenticator.

#### 2. Secondary Authenticators

Secondary authenticators are used for multi-factor authentication (MFA/2FA).

Supported secondary authenticators include:

* Password
* Time-based One-Time Password (TOTP)
* One-Time Passcode (Email) — `oob_otp_email`
* One-Time Passcode (SMS/WhatsApp) — `oob_otp_sms`

Unlike identities, **secondary authenticators do not require uniqueness**.

This allows multiple users to register the same email or phone number as their second-factor method, useful for shared devices or corporate environments.
