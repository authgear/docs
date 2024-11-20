---
description: >-
  Account Lockout Policy safeguards attacks towards a user account from
  brute-force login attempts
---

# Brute-force Protection

Authgear offers brute-force protection by locking the account from login after a number of failed login attempts. Further login attempts can be limited or blocked. This policy can apply to

* An IP address fails multiple times to login as the same user
* The same account fails to login multiple times regardless of IP addresses

The account remains blocked for an amount of "**Lockout Duration**" configured by the admin. The attempt count is reset after a successful login or the "**Reset duration**" passed after the last failure.

To setup the account lockout policy:

1. Go to **Portal** > **Authentication** > **Login Methods**
2. Go to the **Account Lockout Policy** tab
3. Enable the policy
4. Save the config

#### Lockout Threshold

<figure><img src="../.gitbook/assets/image (7).png" alt="" width="563"><figcaption></figcaption></figure>

In this section, you can control after how many failed attempts the lockout will take effects, and the reset duration of the attempts. By default, the lock will be reset after 24 hours has passed from the last failed login attempt.

#### Lockout Duration

<figure><img src="../.gitbook/assets/image (23).png" alt="" width="563"><figcaption></figcaption></figure>

This section determined the amount of time an account will remain locked when it has exceeded the lockout threshold.

In the above example, the lockout duration starts off with 1 minute. For every subsequent failed attempt, lockout duration will increase by 2 times, which means the account will remain locked for 2 minutes after the next attempt, until it reaches the maximum of 60 minutes.

#### Lockout Type

<figure><img src="../.gitbook/assets/image (26).png" alt="" width="563"><figcaption></figcaption></figure>

You can choose to protect against only the attempts from a single IP address or from any devices regardless of the IP address.

#### Apply only to selected authenticators

<figure><img src="../.gitbook/assets/image (11).png" alt="" width="563"><figcaption></figcaption></figure>

The brute-force protection do not only applies to password, it can also protect against other authenticator attempts.
