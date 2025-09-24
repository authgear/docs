---
description: Allow users to authenticate with a custom flow.
---

# Custom Authentication Flow

In Authgear, you can build your own custom login and sign-up pages instead of using the built-in ones.&#x20;

## Example 1: Verifying multiple identities during signup

In **Authentication > Login Methods**, if Email and phone number are both enabled, the user will be able to choose **between** their preferred method during login.&#x20;

<figure><img src="../../.gitbook/assets/Screenshot 2025-08-06 at 15.30.34.png" alt=""><figcaption></figcaption></figure>

In the example above, the user can choose to signup/login with **either** an email address **or** a phone number, but they cannot register with both at the same time.&#x20;

One popular signup flow is to require the user to verify both their email and phone during signup, allowing them to log in with either in the future. This can be enabled by adding the following configuration to the YAML in **Portal >** **Advanced >** **Edit Config**.&#x20;

{% code lineNumbers="true" %}
```yaml
authentication_flow:
  signup_flows: # signup_flows, login_flows, signup_login_flows, reauth_flows, or account_recovery_flows
  - name: default # the flow that is shown first to the user
    steps:
    - name: setup_email
      type: identify # action type: verify, identify, create_authenticator
      one_of:
      - identification: email
        steps:
        - type: verify
          target_step: setup_email
        - type: create_authenticator
          name: authenticate_primary_email
          one_of: 
          - authentication: primary_oob_otp_email
            target_step: setup_email
    - name: setup_phone
      type: identify
      one_of:
      - identification: phone
        steps:
        - type: verify
          target_step: setup_phone
        - type: create_authenticator
          name: authenticate_primary_phone
          one_of:
          - authentication: primary_oob_otp_sms
            target_step: setup_phone          
```
{% endcode %}

The two main steps of this signup process are `setup_email`, then  `setup_phone`. The action `type` of both steps is `identify`.

Each step has allowed identification methods (e.g. `email`, `phone`)

* Each identification method has `steps` defining actions to perform, which are:
  * `verify`: confirms ownership of the credential
  * `create_authenticator`: registers the credential as a valid login method
    * `authentication` defines **how** the user will be authenticated. For example, using the `primary_oob_otp_sms`
  * `target_step`: The YAML structure requires certain nested steps to explicitly reference the parent step.
* `one_of` is used in some steps to define branching options. To adhere to the syntax, `one_of` must be included even if there is only one option available.

In this example, a sign up flow is defined without the login steps (see Line:2), the "Automatically signup a new user if a login ID is not found during login" option should be disabled in the portal.&#x20;

<figure><img src="../../.gitbook/assets/Screenshot 2025-09-01 at 09.27.32.png" alt=""><figcaption></figcaption></figure>

To further modify the authflow to your needs, you can refer to the [specification of Auth Flow](https://github.com/authgear/authgear-server/blob/main/docs/specs/authentication-flow.md), and the [API reference for Auth Flow API](https://github.com/authgear/authgear-server/blob/main/docs/specs/authentication-flow-api-reference.md).&#x20;
