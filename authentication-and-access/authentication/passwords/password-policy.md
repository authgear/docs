---
description: Setting format and strength requirements for passwords
---

# Password Policy

Authgear allows you to set a password policy for your project. This page walks through setting password requirements, password strength, keywords to exclude, and password expiry from the Authgear Portal.

You can configure your password policy in password settings.

## 1. Password Requirements

Choose a minimum character length, and use the checkboxes provided to include one or more requirements for a valid password.&#x20;

<figure><img src="../../../.gitbook/assets/Screenshot 2025-08-05 at 15.22.53.png" alt=""><figcaption></figcaption></figure>

To ensure your updated password policy applies to both existing and new users, toggle on "Force password change on next login". All users will be required to update their passwords if their current passwords do not meet the newly configured policy.&#x20;

## 2. Password Strength

#### What is password strength?

Password strength is simply a measure of how difficult it is to guess or crack a password.&#x20;

Authgear currently uses the [zxcvbn](https://github.com/dropbox/zxcvbn) password strength estimator library, which goes beyond basic requirements (like length or character variety) and uses pattern matching to recognize common insecure passwords.&#x20;

<details>

<summary>How password strength is calculated in Authgear</summary>

A password is scored for how uncommon and guessable it is using the zxcvbn algorithm.

The following table shows the scores for the various minimum password strength levels in Authgear.

| Password Strength Level | Score | Description                                                                                                          |
| ----------------------- | ----- | -------------------------------------------------------------------------------------------------------------------- |
| N/A                     | -     | Totally ignore the Advance password strength score and use the Basic password policy. E.g. Minimum password length.  |
| Extremely guessable     | 0     | Too guessable: risky password. (guesses < 10^3)                                                                      |
| Very guessable          | 1     | Very guessable: protection from throttled online attacks. (guesses < 10^6)                                           |
| Fair                    | 2     | <p>Somewhat guessable: protection from unthrottled online attacks. (guesses &#x3C; 10^8)</p><p></p>                  |
| Very unguessable        | 3     | <p>Safely unguessable: moderate protection from offline slow-hash scenario. (guesses &#x3C; 10^10)</p><p></p>        |
| Extremely unguessable   | 4     | Very unguessable: strong protection from offline slow-hash scenario. (guesses >= 10^10)                              |



</details>

### How to set password strength for your Authgear project

Scroll down to the Advanced sub-section of the Password tab, then click select your preferred option from the **Min. password strength level** dropdown.

<figure><img src="../../../.gitbook/assets/Screenshot 2025-08-05 at 15.35.39.png" alt=""><figcaption></figcaption></figure>



## 3. Prevent Password Reuse

Toggle on **Prevent Password Reuse** to ensure a new, unique password is set during password changes.

In the following example, the new password cannot match any password used within the 90 days, or any last 3 previously used passwords.

<figure><img src="../../../.gitbook/assets/Screenshot 2025-08-05 at 15.37.33.png" alt=""><figcaption></figcaption></figure>



## 4. Keywords to be Excluded from Password

You can also disallow specific keywords in the user's password. Simply add them to the "Keywords to be excluded" field, and the admin or user will not be able to set a password containing the listed keywords.&#x20;

<figure><img src="../../../.gitbook/assets/Screenshot 2025-08-03 at 20.10.50.png" alt=""><figcaption></figcaption></figure>

## 5. Password Expiry

See [password-expiry.md](password-expiry.md "mention")



Once you're done, remember to hit **Save** to keep your changes.
