---
description: Blocking events are triggered before the operation is performed.
---

# Blocking Events

Blocking events are triggered before the operation is performed, such as before user creation. This allows you to abort or alter the operations programmatically.

They are delivered to your hooks synchronously, right before changes are persisted to the database.

The list of events:

* [user.pre\_create](blocking-events.md#user.pre_create)
* [user.profile.pre\_update](blocking-events.md#user.profile.pre_update)
* [user.pre\_schedule\_deletion](blocking-events.md#user.pre_schedule_deletion)
* [user.pre\_schedule\_anonymization](blocking-events.md#user.pre_schedule_anonymization)
* [authentication.pre\_initialize](blocking-events.md#authentication.pre_initialize)
* [authentication.post\_identified](blocking-events.md#authentication.post_identified)
* [authentication.pre\_authenticated](blocking-events.md#authentication.pre_authenticated)
* [oidc.jwt.pre\_create](blocking-events.md#oidc.jwt.pre_create)

Your hooks must return a JSON document to indicate whether the operation should continue. For example, to let the operation proceed, return the following JSON:

```json5
{
  "is_allowed": true
}
```

Each of your hooks must respond within 5 seconds. All of your hooks must complete within 10 seconds. Otherwise, the delivery will fail due to timeout.

Supported fields in the JSON response:

<table><thead><tr><th width="178.265625">Parameter</th><th width="259.9609375">Description</th><th>Supported Blocking events</th></tr></thead><tbody><tr><td><code>is_allowed</code></td><td>Allow or abort the current operation</td><td>All blocking events</td></tr><tr><td><code>reason</code></td><td>(When <code>is_allowed</code> is false) A human-readable explanation for rejecting the operation. This will be shown in the UI. </td><td>All blocking events</td></tr><tr><td><code>title</code></td><td>(When <code>is_allowed</code> is false)  A human-readable title for rejecting the operation. This will be shown in the UI. </td><td>All blocking events</td></tr><tr><td><code>mutation</code></td><td>Mutate the object in the Event payload.</td><td>user.pre_create<br>user.profile.pre_update<br>user.pre_schedule_deletion user.pre_schedule_anonymization<br>oidc.jwt.pre_create</td></tr><tr><td><code>constraints</code></td><td>Apply authentication constraints that are required for the authentication. For example, require Email OTP 2FA for this request.</td><td>authentication.pre_initialize<br>authentication.post_identified<br>authentication.pre_authenticated</td></tr><tr><td><code>rate_limits</code></td><td>Dynamically override rate limit weights.</td><td>authentication.pre_initialize<br>authentication.post_identified<br>authentication.pre_authenticated</td></tr><tr><td><code>bot_protection</code></td><td>Determines whether bot protection should be enabled for this request.</td><td>authentication.pre_initialize<br>authentication.post_identified</td></tr></tbody></table>

## Deny Access with "is\_allowed"

To let the operation proceed, return a JSON document with `is_allowed` set to `true`.

```json5
{
  "is_allowed": true
}
```

To abort the operation, return a JSON document with `is_allowed` set to `false`, and a non-empty `reason` and `title`.

```json5
{
  "is_allowed": false,
  "reason": "some reason",
  "title": "some title"
}
```

If any of your hooks abort the operation, the operation is aborted. The `reason` and `title` will be shown to the end-user as an error message.

## Mutations

Your hooks can optionally apply a mutation for certain blocking events. The supported mutation is specific to each type of blocking event. Refer to the [Event List](non-blocking-events.md) to see what mutations are supported.

Mutations by a hook are applied only when the operation is allowed to proceed, and take effect only when all hooks allow the operation to proceed.

* Objects not appearing in `mutations` are left intact.&#x20;
* The mutated objects are **NOT** merged with the original ones.
* The mutated objects are **NOT** validated, and are propagated along the hook chain. The mutated objects are validated after traversing the hook chain.
* Mutations do **NOT** generate extra events to avoid infinite loops.

### Mutations on the user object

When a blocking event supports mutations on the user object, your hooks can respond with a JSON document to allow the operation, and specify the mutations you want to apply on the user object.

```json5
{
  "is_allowed": true,
  "mutations": {
    "user": {
      "standard_attributes": {
        "name": "John"
      },
      "custom_attributes": {
        "age": 30
      }
    }
  }
}
```

To mutate the user object, include `user` inside `mutations`. Only `standard_attributes` , `custom_attributes` , `roles` and `groups` of the user object are mutable.

{% hint style="info" %}
You must include the **WHOLE** `standard_attributes` , `custom_attributes` , `roles` or `groups` when you specify the mutations. Otherwise, missing attributes **WILL BE** deleted.
{% endhint %}

### Mutations on the JWT payload

If a blocking event supports mutations on the JWT payload, your hooks can respond with a JSON document that allows the operation, and specify additional fields that you want to include in the JWT payload. However, you **MUST NOT** change or remove any existing fields in the JWT payload, as they are essential to the validity of the JWT.

```json
{
  "is_allowed": true,
  "mutations": {
    "jwt": {
      "payload": {
        // The original payload you get from the event object.
        "iss": "https://myapp.authgear.cloud",
        "aud": ["YOUR_CLIENT_ID"],
        "sub": "THE_USER_ID",
        // Other essential JWT fields that you MUST retain.

        // Additional fields that you want to add.
        "https://myapp.com": {
          "custom_field": "custom_value"
        }
      }
    }
  }
}
```

To add additional fields to the JWT payload, include the **WHOLE** `jwt.payload` inside `mutations`. You must only add your own custom fields.&#x20;

## Apply Authentication Constraints

Use the `constraints` property to conditionally require additional authentication methods for the current request.

* `amr`: An array of Authentication Methods References (AMR) values that are required for the authentication. The supported values are:
  * `pwd`
  * `otp`
  * `sms`
  * `mfa`
  * `x_primary_password`
  * `x_primary_oob_otp_email`
  * `x_primary_oob_otp_sms`
  * `x_secondary_password`
  * `x_secondary_oob_otp_email`
  * `x_secondary_oob_otp_sms`
  * `x_secondary_totp`

When multiple values are returned in `amr`, they represent a combined requirement where user must satisfy all listed methods. For example, for `"amr": ["mfa", "otp"]`, the user must fulfil `mfa` AND `otp` in the authentication flow.

Example response for requiring MFA in authentication:

```json
{
  "is_allowed": true,
  "constraints": {
    "amr": ["mfa"]
  }
}
```

The constraints are enforced based on the authentication flow type:

* Signup / Promote: Enforces the setup of corresponding authenticator according to the value of `amr`.
* Login / Re-authentication: Enforces the use of corresponding authenticator according to the value of `amr`.
* Account Recovery: No effect (does not support 2FA)

## Override Rate Limits

Use the `rate_limits` property in a blocking event response to dynamically override rate limit weights. The `weight` parameter defines the amount of value contributed to the specified rate limit in subsequent operations. The default value is 1. A value of 0 means that subsequent operations will not be counted towards the rate limit.

For example, this hook response is saying that, in the next operation, the "account enumeration" rate limit will be double counted. i.e. The user shall hit the rate limit with half the number of attempts.

```json
{
  "is_allowed": true,
  "rate_limits": {
    "authentication.account_enumeration": {
      "weight": 2
    }
  }
}
```

The following rate limits are supported:

* `authentication.general` : Verify any credentials, e.g. password, OTP
* `authentication.account_enumeration` : Check existence of a login ID (username, email, phone number)

## Bot Protection

Use `bot_protection` property to enable or disable for this request. This overrides the project's configuration.

Example response with bot protection override:

```json
{
  "is_allowed": true,
  "bot_protection": {
    "mode": "always"
  }
}
```

This overrides the original `mode` of bot\_protection in your config. Supported values are:

* `always` : Require Captcha
* `never` : Disable Captcha

## Blocking Event List

### user.pre\_create

Occurs right before the user creation. A user can be created by user signup, user signup as an anonymous user, or created by the admin via the Portal or Admin API.

This event supports [#mutations-on-the-user-object](blocking-events.md#mutations-on-the-user-object "mention")

```json
{
  "type": "user.pre_create",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    },
    "identities": [
      {
        "id": "239d585d-9b90-4148-9aa2-2e3131b5847a",
        "created_at": "2006-01-02T03:04:05.123456Z",
        "updated_at": "2006-01-02T03:04:05.123456Z",
        "type": "login_id",
        "claims": {
          "email": "user@example.com",
          "https://authgear.com/claims/login_id/key": "email",
          "https://authgear.com/claims/login_id/original_value": "user@example.com",
          "https://authgear.com/claims/login_id/type": "email",
          "https://authgear.com/claims/login_id/value": "user@example.com"
        }
      }
    ]
  }
}
```

### user.profile.pre\_update

Occurs right before the update of the user profile.

This event supports [#mutations-on-the-user-object](blocking-events.md#mutations-on-the-user-object "mention")

```json
{
  "type": "user.profile.pre_update",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "name": "Chris",
        "updated_at": 1136171045
      }
    }
  }
}
```

### user.pre\_schedule\_deletion

Occurs right before account deletion is scheduled.

This event supports [#mutations-on-the-user-object](blocking-events.md#mutations-on-the-user-object "mention")

```json
{
  "type": "user.pre_schedule_deletion",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": true,
      "is_deactivated": true,
      "delete_at": "2022-09-30T15:18:19.040081Z",
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    }
  }
}
```

### **user.pre\_schedule\_anonymization**

Occurs right before the account anonymization is scheduled.

This event supports [#mutations-on-the-user-object](blocking-events.md#mutations-on-the-user-object "mention")

```json
{
  "type": "user.pre_schedule_anonymization",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": true,
      "is_deactivated": true,
      "delete_at": "2022-09-30T15:18:19.040081Z",
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    }
  }
}
```

### authentication.pre\_initialize

Occurs right before any authentication flow started.

This event supports [#apply-authentication-constraints](blocking-events.md#apply-authentication-constraints "mention"), [#bot-protection](blocking-events.md#bot-protection "mention"), [#override-rate-limits](blocking-events.md#override-rate-limits "mention")

```json
{
  "type": "authentication.pre_initialize",
  "payload": {
    "authentication_context": {
      "user": null,
      "asserted_authentications": [],
      "asserted_identifications": [],
      "amr": [],
      "authentication_flow": {
        "type": "login",
        "name": "default"
      }
    }
  }
}
```

* The `authentication_context` object contains details about the current authentication.

### **authentication.post\_identified**

Occurs right after an identity is identified during authentication, for example after user enters their email address, username or phone number.

This event supports [#apply-authentication-constraints](blocking-events.md#apply-authentication-constraints "mention"), [#bot-protection](blocking-events.md#bot-protection "mention"), [#override-rate-limits](blocking-events.md#override-rate-limits "mention")

```json
{
  "type": "authentication.post_identified",
  "payload": {
    "authentication_context": {
      "authentication_flow": {
        "type": "login",
        "name": "default"
      },
      "user": { /* The identified user */
        "id": "c1397fc7-10ff-4cbd-bdc9-6fd9ae829c86",
        "is_anonymized": false,
        "is_anonymous": false,
        "is_deactivated": false,
        "is_disabled": false,
        "is_verified": true,
        "last_login_at": "2025-05-27T06:32:54.072273Z",
        "roles": [],
        "groups": [],
        "standard_attributes": {
          "email": "user@example.com",
          "email_verified": true
        },
        "custom_attributes": {},
        "created_at": "2025-05-27T06:32:54.005206Z",
        "updated_at": "2025-05-27T06:32:54.066087Z"
      },
      "asserted_identifications": [
        { /* The identification methods asserted in the current authentication */
          "identification": "oauth",
          "identity": {
            "id": "8f84ed75-5c8b-45c1-b657-b0c65ac3affe",
            "claims": {
              "email": "user@example.com",
              "email_verified": true,
              "family_name": "Authgear",
              "given_name": "Test",
              "https://authgear.com/claims/oauth/provider_alias": "google",
              "https://authgear.com/claims/oauth/provider_type": "google",
              "https://authgear.com/claims/oauth/subject_id": "1234567",
              "name": "Test Authgear"
            },
            "type": "oauth",
            "created_at": "2025-05-27T06:32:54.02264Z",
            "updated_at": "2025-05-27T06:32:54.02264Z"
          }
        }
      ],
      "asserted_authentications": [],
      "amr": []
    },
    "identification": {
      "identification": "oauth",
      "identity": { /* The identified identity */
        "id": "8f84ed75-5c8b-45c1-b657-b0c65ac3affe",
        "claims": {
          "email": "user@example.com",
          "email_verified": true,
          "family_name": "Authgear",
          "given_name": "Test",
          "https://authgear.com/claims/oauth/provider_alias": "google",
          "https://authgear.com/claims/oauth/provider_type": "google",
          "https://authgear.com/claims/oauth/subject_id": "1234567",
          "name": "Test Authgear"
        },
        "type": "oauth",
        "created_at": "2025-05-27T06:32:54.02264Z",
        "updated_at": "2025-05-27T06:32:54.02264Z"
      }
    }
  }
}
```

* The `authentication_context` object contains details about the current authentication.
* The `identification` object contains details about the identity entered and identified.

### authentication.pre\_authenticated

Occurs right before any authentication flows completed. It is triggered when the remaining flow no longer contains any of the steps: `identify` / `authenticate` / `create_authenticator` / `verify`

This event supports [#apply-authentication-constraints](blocking-events.md#apply-authentication-constraints "mention"), [#override-rate-limits](blocking-events.md#override-rate-limits "mention")

```json
{
  "payload": {
    "authentication_context": {
      "authentication_flow": { 
        "type": "login",
        "name": "default"
      },
      "user": { /* The identified user */
        "id": "c1397fc7-10ff-4cbd-bdc9-6fd9ae829c86",
        "is_anonymized": false,
        "is_anonymous": false,
        "is_deactivated": false,
        "is_disabled": false,
        "is_verified": true,
        "last_login_at": "2025-05-27T06:32:54.072273Z",
        "roles": [],
        "groups": [],
        "standard_attributes": {
          "email": "user@example.com",
          "email_verified": true
        },
        "custom_attributes": {},
        "created_at": "2025-05-27T06:32:54.005206Z",
        "updated_at": "2025-05-27T06:32:54.066087Z"
      },
      "asserted_identifications": [ /* The identification methods asserted in the current authentication */
        {
          "identification": "oauth",
          "identity": {
            "id": "8f84ed75-5c8b-45c1-b657-b0c65ac3affe",
            "claims": {
              "email": "user@example.com",
              "email_verified": true,
              "family_name": "Authgear",
              "given_name": "Test",
              "https://authgear.com/claims/oauth/provider_alias": "google",
              "https://authgear.com/claims/oauth/provider_type": "google",
              "https://authgear.com/claims/oauth/subject_id": "1234567",
              "name": "Test Authgear"
            },
            "type": "oauth",
            "created_at": "2025-05-27T06:32:54.02264Z",
            "updated_at": "2025-05-27T06:32:54.02264Z"
          }
        },
      ],
      "asserted_authentications": [ /* Authentication methods asserted during the authentication */
        {
          "authentication": "primary_oob_otp_sms",
          "authenticator": {
            "id": "2a6f9927-c76c-4112-868a-879547239266",
            "type": "oob_otp_sms",
            "kind": "primary"
          }
        }
      ],
      "amr": ["sms", "otp", "x_primary_oob_otp_sms"]
    },
  }
}
```

* The `authentication_context` object contains details about the current authentication.

### oidc.jwt.pre\_create

Occurs right before the access token is issued. Use this event to add custom fields to the JWT access token.

This event supports [#mutations-on-the-jwt-payload](blocking-events.md#mutations-on-the-jwt-payload "mention")

```json
{
  "type": "oidc.jwt.pre_create",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": true,
      "is_deactivated": true,
      "delete_at": "2022-09-30T15:18:19.040081Z",
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    },
    "jwt": {
      "payload": {
        "iss": "https://myapp.authgear.cloud",
        "aud": ["YOUR_CLIENT_ID"],
        "sub": "338deafa-400b-4589-a922-2c92d670b757"
      }
    }
  }
}
```
