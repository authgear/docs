---
description: >-
  Connect a social or enterprise login to an existing account that shares the
  same email, instead of creating a duplicate user or rejecting the signup.
---

# Account Linking

When you add social login to an app that already has users, some of them will click **Continue with Google** even though they originally signed up with an email and password. Both identities carry the same email address, but by default Authgear treats the Google signup as an attempt to create a second account and rejects it with an "account already exists" error.

Account linking changes what happens next. Instead of an error, Authgear detects that the incoming Google account matches an existing user, asks the user to log in to that account once to prove they own it, then attaches the Google identity to it. The user ends up with one account and two ways to sign in.

{% hint style="info" %}
Account linking runs automatically **during signup**. To let an already signed-in user connect a provider from your own UI, see [Link and Unlink Social/Enterprise Provider with the SDK](link-and-unlink-social-enterprise-provider-with-the-sdk.md). Users can also manage their connections on the [User Settings](../../customization/ui-customization/built-in-ui/user-settings.md) page.
{% endhint %}

## When to use it

Enable account linking when the same person can reach your app through more than one signup method and you want them to keep a single account. Typical situations:

* **You added Google login to an app with existing password users.** A user who signed up with `alice@example.com` and a password clicks **Continue with Google**. Her Google account uses the same email. With account linking she logs in with her password once, and from then on can sign in with either method.
* **You offer several social providers.** A user signs up with Google today and taps **Continue with Apple** on their phone next month. Both accounts share one email; linking keeps them on one Authgear account instead of two.
* **You migrated users into Authgear and later enabled an enterprise provider.** Imported users match their Microsoft Entra ID or ADFS accounts by email, so their first enterprise login attaches to the account you imported rather than creating a fresh one.

## How it works

When a user signs up with a social or enterprise provider, Authgear reads a claim from the provider's user profile (the email, by default) and compares it against the identities of existing users. What happens on a match depends on the `action` you configure:

| Action           | Behavior                                                                                                                                                                      |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `error`          | Reject the signup with a "duplicated identity" error. This is the default when account linking is not configured.                                                             |
| `login_and_link` | Show the user their existing account and ask them to log in to it. After they authenticate, add the new provider identity to that account and continue the signup. |

{% hint style="warning" %}
The login step in `login_and_link` prevents account takeover. Some identity providers let anyone register with an unverified email, so an email match alone doesn't prove ownership — an attacker could register `alice@example.com` at such a provider and hijack Alice's account. The login does prove it.
{% endhint %}

## Enable account linking

This walkthrough uses Google, but the same configuration works for any provider you have set up.

### Prerequisites

* A Google login provider configured in the Portal under **Authentication > Social / Enterprise Login**, with an alias. This guide assumes the alias `google`. See [Connect Apps to Google](social-login-providers/google.md) if you haven't set this up.

### Add the configuration

You configure account linking in your project's YAML config. In the Portal, go to **Advanced > Edit Config** and add an `account_linking` section:

```yaml
account_linking:
  oauth:
    - alias: google
      oauth_claim:
        pointer: "/email"
      user_profile:
        pointer: "/email"
      action: login_and_link
```

Save the config. The change takes effect immediately.

Each entry under `account_linking.oauth` defines one linking rule:

| Field          | Meaning                                                                                                                                                    |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `alias`        | Which provider this rule applies to. Must match the alias of a provider under **Social / Enterprise Login**.                                                |
| `oauth_claim`  | A JSON pointer to the claim to read from the incoming provider profile. `/email` reads the email Google reports.                                            |
| `user_profile` | A JSON pointer to the attribute to match against existing users' identities. `/email` matches users whose email login ID or standard attribute is the same. |
| `action`       | What to do on a match: `error` or `login_and_link`.                                                                                                          |

{% hint style="info" %}
`oauth_claim` and `user_profile` accept only `/email`, `/phone_number`, and `/preferred_username`. Matching by email covers Google and most social providers; `/preferred_username` is useful for enterprise providers such as ADFS that identify users by username.
{% endhint %}

### What the user experiences

With the config above in place, here is the full flow for a user who already has a password account under `alice@example.com`:

1. On the signup page, the user clicks **Continue with Google** and completes Google's consent screen.
2. Authgear finds the existing account and shows an **Existing account found** page listing it with a masked identifier, such as `al***@example.com`.
3. The user selects the account and logs in with their existing method — here, their password.
4. Authgear links the Google identity to the account. If your signup flow requires steps the account doesn't satisfy yet (for example, setting up two-factor authentication), the user completes them now; steps the account already satisfies are skipped.
5. The user is signed in. From now on, both the password and Google sign the user in to the same account.

## Linking email, phone, or username signups

Account linking also covers the reverse direction: a user who signed up with Google first, then tries to sign up with the same email address and a password. Configure it under `account_linking.login_id`:

```yaml
account_linking:
  login_id:
    - key: email
      user_profile:
        pointer: "/email"
      action: login_and_link
```

`key` names the login ID type (`email`, `phone`, or `username` by default). When a signup with that login ID matches an existing user, the user logs in with the existing account — in this case, through Google — and the new email login ID is added to it. Without this config, the conflicting signup fails with an error, as before.

## Advanced: different rules per signup flow

If you define [custom authentication flows](../authentication/custom-authentication-flow.md), you can override a linking rule inside a specific signup flow step. Give the rule a `name` in `account_linking`, then reference that name in the step to override its `action` or pick which login flow handles the login:

```yaml
account_linking:
  oauth:
    - name: google_linking
      alias: google
      oauth_claim:
        pointer: "/email"
      user_profile:
        pointer: "/email"
      action: error
authentication_flow:
  signup_flows:
    - name: customer_signup
      steps:
        - name: identify
          type: identify
          one_of:
            - identification: oauth
              account_linking:
                oauth:
                  - name: google_linking
                    action: login_and_link
                    login_flow: customer_login
```

Here Google linking fails with an error everywhere except the `customer_signup` flow, which allows it and runs the `customer_login` flow for the login step. When `login_flow` is not set, Authgear uses the login flow with the same name as the current signup flow.

## Account linking in Custom UI

If you build your own signup UI with the [Authentication Flow API](../../customization/ui-customization/custom-ui/authentication-flow-api.md), account linking appears as an `identify` action whose `data.type` is `account_linking_identification_data`. The data contains the list of matched accounts with masked display names; your UI submits the index of the account the user picks, and the flow continues into the login steps. See the [Authentication Flow API reference](https://github.com/authgear/authgear-server/blob/main/docs/specs/authentication-flow-api-reference.md) for the exact schema.
