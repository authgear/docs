---
description: The full list of events
---

# Event List

## Blocking Events

* [user.pre\_create](event-list.md#user.pre\_create)
* [user.profile.pre\_update](event-list.md#user.profile.pre\_update)
* [user.pre\_schedule\_deletion](event-list.md#user.pre\_schedule\_deletion)
* [oidc.jwt.pre\_create](event-list.md#oidc.jwt.pre\_create)

## Non-blocking Events

* [user.created](event-list.md#user.created)
* [user.profile.updated](event-list.md#user.profile.updated)
* [user.authenticated](event-list.md#user.authenticated)
* [user.disabled](event-list.md#user.disabled)
* [user.reenabled](event-list.md#user.reenabled)
* [user.anonymous.promoted](event-list.md#user.anonymous.promoted)
* [user.deletion\_scheduled](event-list.md#user.deletion\_scheduled)
* [user.deletion\_unscheduled](event-list.md#user.deletion\_unscheduled)
* [user.deleted](event-list.md#user.deleted)
* [identity.email.added](event-list.md#identity.email.added)
* [identity.email.removed](event-list.md#identity.email.removed)
* [identity.email.updated](event-list.md#identity.email.updated)
* [identity.email.verified](event-list.md#identity.email.verified)
* [identity.email.unverified](event-list.md#identity.email.unverified)
* [identity.phone.added](event-list.md#identity.phone.added)
* [identity.phone.removed](event-list.md#identity.phone.removed)
* [identity.phone.updated](event-list.md#identity.phone.updated)
* [identity.phone.verified](event-list.md#identity.phone.verified)
* [identity.phone.unverified](event-list.md#identity.phone.unverified)
* [identity.username.added](event-list.md#identity.username.added)
* [identity.username.removed](event-list.md#identity.username.removed)
* [identity.username.updated](event-list.md#identity.username.updated)
* [identity.oauth.connected](event-list.md#identity.oauth.connected)
* [identity.oauth.disconnected](event-list.md#identity.oauth.disconnected)
* [identity.biometric.enabled](event-list.md#identity.biometric.enabled)
* [identity.biometric.disabled](event-list.md#identity.biometric.disabled)

### user.pre\_create

Occurs right before the user creation. User can be created by user signup, user signup as an anonymous user, or created by the admin via the Portal or Admin API.

This event supports [Mutations on the user object](./#mutations-on-the-user-object)

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

This event supports [Mutations on the user object](./#mutations-on-the-user-object)

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

This event does not support mutations.

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

### oidc.jwt.pre\_create

Occurs right before the access token is issued. Use this event to add custom fields to the JWT access token.

This event supports [Mutations on the JWT payload](./#mutations-on-the-jwt-payload)

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

### user.created

Occurs after a new user is created. User can be created by user signup, user signup as an anonymous user, or created by the admin via the Portal or Admin API.

```json
{
  "type": "user.created",
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

### user.profile.updated

Occurs when the user profile is updated.

```json
{
  "type": "user.profile.updated",
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

### user.authenticated

Occurs after the user logged in.

```json
{
  "type": "user.authenticated",
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
        "updated_at": 1136171045
      }
    },
    "session": {
      "id": "6e6e5d9b-7f85-4a8f-a157-2de94694dfea",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "idp",
      "amr": ["pwd"],
      "lastAccessedAt": "2006-01-02T03:04:05.123456Z",
      "createdByIP": "127.0.0.1",
      "lastAccessedByIP": "127.0.0.1",
      "lastAccessedByIPCountryCode": "",
      "lastAccessedByIPEnglishCountryName": "",
      "displayName": "Chrome 104.0.0"
    }
  }
}
```

### user.disabled

Occurs when the user was disabled.

```json5
{
  "type": "user.disabled",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": true,
      "is_deactivated": false,
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

### user.reenabled

Occurs when the user was re-enabled.

```json5
{
  "type": "user.reenabled",
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
        "updated_at": 1136171045
      }
    }
  }
}
```

### user.anonymous.promoted

Occurs whenever an anonymous user is promoted to a normal user.

```json
{
  "type": "user.anonymous.promoted",
  "payload": {
    "anonymous_user": {
      "id": "7a009f88-c636-4245-91ec-7b174dc6a1a1",
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
        "updated_at": 1136171045
      }
    },
    "user": {
      "id": "7a009f88-c636-4245-91ec-7b174dc6a1a1",
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
        "updated_at": 1136171045
      }
    },
    "identities": [
      {
        "id": "1450234b-5e2c-4aa0-b400-f2d9d70a2f45",
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

### user.deletion\_scheduled

Occurs when an account deletion was scheduled.

```json5
{
  "type": "user.deletion_scheduled",
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
      "delete_at": "2006-01-02T03:04:05.123456Z",
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

### user.deletion\_unscheduled

Occurs when an account deletion was unscheduled.

```json5
{
  "type": "user.deletion_unscheduled",
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
        "updated_at": 1136171045
      }
    }
  }
}
```

### user.deleted

Occurs when the user was deleted.

```json5
{
  "type": "user.deleted",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": false,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": false,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": false,
        "updated_at": 1136171045
      }
    }
  }
}
```

### identity.email.added

Occurs when a new email is added to an existing user. Email can be added by the user in the setting page, added by the admin through the Admin API or Portal.

```json
{
  "type": "identity.email.added",
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
        "phone_number": "+447400123456",
        "phone_number_verified": true,
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "a0c55481-147e-4a58-876e-10dffedfd5cd",
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
  }
}
```

### identity.email.removed

Occurs when an email address is removed from an existing user. Email can be removed by the user in the setting page, removed by admin through admin API or Portal.

```json
{
  "type": "identity.email.removed",
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
        "phone_number": "+447400123456",
        "phone_number_verified": true,
        "updated_at": 1136171045
      }
    },
    "identity": {
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
  }
}
```

### identity.email.updated

Occurs when an email address is updated. Email can be updated by the user on the setting page.

```json
{
  "type": "identity.email.updated",
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
        "email": "user3@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    },
    "new_identity": {
      "id": "239d585d-9b90-4148-9aa2-2e3131b5847a",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "email": "user3@example.com",
        "https://authgear.com/claims/login_id/key": "email",
        "https://authgear.com/claims/login_id/original_value": "user3@example.com",
        "https://authgear.com/claims/login_id/type": "email",
        "https://authgear.com/claims/login_id/value": "user3@example.com"
      }
    },
    "old_identity": {
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
  }
}
```

### identity.email.verified

Occurs when an email address is change from unverified to verified for an existing user. Email can be verified by the user in the setting page, mark verified by admin through admin API or Portal.

```json
{
  "type": "identity.email.verified",
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
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "239d585d-9b90-4148-9aa2-2e3131b5847a",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "email": "user@example.com",
        "https://authgear.com/claims/login_id/key": "email",
        "https://authgear.com/claims/login_id/type": "email",
        "https://authgear.com/claims/login_id/value": "user@example.com"
      }
    }
  }
}
```

### identity.email.unverified

Occurs when an email address is unverified. Email can be unverified using Admin API.

```json
{
  "type": "identity.email.unverified",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": false,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": false,
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "239d585d-9b90-4148-9aa2-2e3131b5847a",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "email": "user@example.com",
        "https://authgear.com/claims/login_id/key": "email",
        "https://authgear.com/claims/login_id/type": "email",
        "https://authgear.com/claims/login_id/value": "user@example.com"
      }
    }
  }
}
```

### identity.phone.added

Occurs when a new phone number is added to an existing user. Phone numbers can be added by the user in the setting page, added by admin through admin API or Portal.

```json
{
  "type": "user.phone.added",
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
        "phone_number": "+447400123456",
        "phone_number_verified": true,
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "9fa5668d-a796-4817-93e1-d4096e5966ac",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "phone",
        "https://authgear.com/claims/login_id/original_value": "+447400123456",
        "https://authgear.com/claims/login_id/type": "phone",
        "https://authgear.com/claims/login_id/value": "+447400123456",
        "phone_number": "+447400123456"
      }
    }
  }
}
```

### identity.phone.removed

Occurs when a phone number is removed from an existing user. Phone numbers can be removed by the user on the setting page, removed by admin through admin API or Portal.

```json
{
  "type": "identity.phone.removed",
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
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "9fa5668d-a796-4817-93e1-d4096e5966ac",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "phone",
        "https://authgear.com/claims/login_id/original_value": "+447400123455",
        "https://authgear.com/claims/login_id/type": "phone",
        "https://authgear.com/claims/login_id/value": "+447400123455",
        "phone_number": "+447400123455"
      }
    }
  }
}
```

### identity.phone.updated

Occurs when a phone number is updated. Phone numbers can be updated by the user on the setting page.

```json
{
  "type": "identity.phone.updated",
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
        "phone_number": "+447400123455",
        "phone_number_verified": true,
        "updated_at": 1136171045
      }
    },
    "new_identity": {
      "id": "9fa5668d-a796-4817-93e1-d4096e5966ac",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "phone",
        "https://authgear.com/claims/login_id/original_value": "+447400123455",
        "https://authgear.com/claims/login_id/type": "phone",
        "https://authgear.com/claims/login_id/value": "+447400123455",
        "phone_number": "+447400123455"
      }
    },
    "old_identity": {
      "id": "9fa5668d-a796-4817-93e1-d4096e5966ac",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "phone",
        "https://authgear.com/claims/login_id/original_value": "+447400123456",
        "https://authgear.com/claims/login_id/type": "phone",
        "https://authgear.com/claims/login_id/value": "+447400123456",
        "phone_number": "+447400123456"
      }
    }
  }
}
```

### identity.phone.verified

Occurs when a phone number is change from unverified to verified for an existing user. Phone can be verified by the user in the setting page, mark verified by admin through admin API or Portal.

```json
{
  "type": "identity.phone.verified",
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
        "phone_number": "+447400123455",
        "phone_number_verified": true,
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "9fa5668d-a796-4817-93e1-d4096e5966ac",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "phone",
        "https://authgear.com/claims/login_id/type": "phone",
        "https://authgear.com/claims/login_id/value": "+447400123455",
        "phone_number": "+447400123455"
      }
    }
  }
}
```

### identity.phone.unverified

Occurs when a phone number is unverified. Phone numbers can be unverified using Admin API.

```json
{
  "type": "identity.phone.unverified",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": false,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "phone_number": "+447400123455",
        "phone_number_verified": false,
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "9fa5668d-a796-4817-93e1-d4096e5966ac",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "phone",
        "https://authgear.com/claims/login_id/type": "phone",
        "https://authgear.com/claims/login_id/value": "+447400123455",
        "phone_number": "+447400123455"
      }
    }
  }
}
```

### identity.username.added

Occurs when a new username is added to an existing user. Username can be added by the user in setting page, added by admin through Admin API or Portal.

```json
{
  "type": "identity.username.added",
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
        "preferred_username": "user01",
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "38683500-f6ce-477d-b944-1f915a451995",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "username",
        "https://authgear.com/claims/login_id/original_value": "user01",
        "https://authgear.com/claims/login_id/type": "username",
        "https://authgear.com/claims/login_id/value": "user01",
        "preferred_username": "user01"
      }
    }
  }
}
```

### identity.username.removed

Occurs when the username is removed from an existing user. The username can be removed by the user on the setting page, removed by admin through admin API or Portal.

```json
{
  "type": "identity.username.removed",
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
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "38683500-f6ce-477d-b944-1f915a451995",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "username",
        "https://authgear.com/claims/login_id/original_value": "user02",
        "https://authgear.com/claims/login_id/type": "username",
        "https://authgear.com/claims/login_id/value": "user02",
        "preferred_username": "user02"
      }
    }
  }
}
```

### identity.username.updated

Occurs when the username is updated. The username can be updated by the user on the setting page.

```json
{
  "type": "identity.username.updated",
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
        "preferred_username": "user02",
        "updated_at": 1136171045
      }
    },
    "new_identity": {
      "id": "38683500-f6ce-477d-b944-1f915a451995",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "username",
        "https://authgear.com/claims/login_id/original_value": "user02",
        "https://authgear.com/claims/login_id/type": "username",
        "https://authgear.com/claims/login_id/value": "user02",
        "preferred_username": "user02"
      }
    },
    "old_identity": {
      "id": "38683500-f6ce-477d-b944-1f915a451995",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "username",
        "https://authgear.com/claims/login_id/original_value": "user01",
        "https://authgear.com/claims/login_id/type": "username",
        "https://authgear.com/claims/login_id/value": "user01",
        "preferred_username": "user01"
      }
    }
  }
}
```

### identity.oauth.connected

Occurs when a user has connected to a new OAuth provider.

```json
{
  "type": "identity.oauth.connected",
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
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "f22425cf-68b2-45f8-936d-3c54c9cdf5c7",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "oauth",
      "claims": {
        "email": "xxx@gmail.com",
        "email_verified": true,
        "family_name": "",
        "given_name": "",
        "https://authgear.com/claims/oauth/profile": {
          "at_hash": "",
          "aud": ["xxx.apps.googleusercontent.com"],
          "azp": "xxx.apps.googleusercontent.com",
          "email": "xxx@gmail.com",
          "email_verified": true,
          "exp": "2006-01-02T03:04:05Z",
          "family_name": "",
          "given_name": "",
          "iat": "2006-01-02T03:04:05Z",
          "iss": "https://accounts.google.com",
          "locale": "en",
          "name": "",
          "nonce": "",
          "picture": "https://lh3.googleusercontent.com/a/xxx",
          "sub": ""
        },
        "https://authgear.com/claims/oauth/provider_alias": "google",
        "https://authgear.com/claims/oauth/provider_type": "google",
        "https://authgear.com/claims/oauth/subject_id": "",
        "locale": "en",
        "name": "",
        "picture": "https://lh3.googleusercontent.com/a/xxx"
      }
    }
  }
}
```

### identity.oauth.disconnected

Occurs when a user is disconnected from an OAuth provider. It can be done by the user disconnecting their OAuth provider in the setting page, or the admin removing the OAuth identity through admin API or Portal.

```json
{
  "type": "identity.oauth.disconnected",
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
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "f22425cf-68b2-45f8-936d-3c54c9cdf5c7",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "oauth",
      "claims": {
        "email": "xxx@gmail.com",
        "email_verified": true,
        "family_name": "",
        "given_name": "",
        "https://authgear.com/claims/oauth/profile": {
          "at_hash": "",
          "aud": ["xxx.apps.googleusercontent.com"],
          "azp": "xxx.apps.googleusercontent.com",
          "email": "xxx@gmail.com",
          "email_verified": true,
          "exp": "2006-01-02T03:04:05Z",
          "family_name": "",
          "given_name": "",
          "iat": "2006-01-02T03:04:05Z",
          "iss": "https://accounts.google.com",
          "locale": "en",
          "name": "",
          "nonce": "",
          "picture": "https://lh3.googleusercontent.com/a/xxx",
          "sub": ""
        },
        "https://authgear.com/claims/oauth/provider_alias": "google",
        "https://authgear.com/claims/oauth/provider_type": "google",
        "https://authgear.com/claims/oauth/subject_id": "",
        "locale": "en",
        "name": "",
        "picture": "https://lh3.googleusercontent.com/a/xxx"
      }
    }
  }
}
```

### identity.biometric.enabled

Occurs when the user enabled biometric login.

```json
{
  "type": "identity.biometric.enabled",
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
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "5cb77960-634b-4c0e-8a0e-6c2c73fb8f47",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "biometric",
      "claims": {
        "https://authgear.com/claims/biometric/device_info": {
          "ios": {
            "NSBundle": {
              "CFBundleDisplayName": "Authgear demo iOS",
              "CFBundleExecutable": "ios_example",
              "CFBundleIdentifier": "com.authgear.exampleapp.ios",
              "CFBundleName": "ios_example",
              "CFBundleShortVersionString": "1.0",
              "CFBundleVersion": "1653565975"
            },
            "NSProcessInfo": {
              "isMacCatalystApp": false,
              "isiOSAppOnMac": false
            },
            "UIDevice": {
              "model": "iPhone",
              "name": "iPhone",
              "systemName": "iOS",
              "systemVersion": "16.1.1",
              "userInterfaceIdiom": "phone"
            },
            "uname": {
              "machine": "iPhone13,3",
              "nodename": "Users-iPhone",
              "release": "22.1.0",
              "sysname": "Darwin",
              "version": "Darwin Kernel Version 22.1.0: Thu Oct  6 19:34:22 PDT 2022; root:xnu-8792.42.7~1/RELEASE_ARM64_T8101"
            }
          }
        },
        "https://authgear.com/claims/biometric/formatted_device_info": "iPhone 12 Pro",
        "https://authgear.com/claims/biometric/key_id": "1CC9D95A-6578-4557-8279-C4D5699D3549"
      }
    }
  }
}
```

### identity.biometric.disabled

Occurs when biometric login is disabled. It will be triggered only when the user disabled it from the settings page or the admin disabled it from the Admin API or portal.

```json
{
  "type": "identity.biometric.disabled",
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
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "5cb77960-634b-4c0e-8a0e-6c2c73fb8f47",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "biometric",
      "claims": {
        "https://authgear.com/claims/biometric/device_info": {
          "ios": {
            "NSBundle": {
              "CFBundleDisplayName": "Authgear demo iOS",
              "CFBundleExecutable": "ios_example",
              "CFBundleIdentifier": "com.authgear.exampleapp.ios",
              "CFBundleName": "ios_example",
              "CFBundleShortVersionString": "1.0",
              "CFBundleVersion": "1653565975"
            },
            "NSProcessInfo": {
              "isMacCatalystApp": false,
              "isiOSAppOnMac": false
            },
            "UIDevice": {
              "model": "iPhone",
              "name": "iPhone",
              "systemName": "iOS",
              "systemVersion": "16.1.1",
              "userInterfaceIdiom": "phone"
            },
            "uname": {
              "machine": "iPhone13,3",
              "nodename": "Users-iPhone",
              "release": "22.1.0",
              "sysname": "Darwin",
              "version": "Darwin Kernel Version 22.1.0: Thu Oct  6 19:34:22 PDT 2022; root:xnu-8792.42.7~1/RELEASE_ARM64_T8101"
            }
          }
        },
        "https://authgear.com/claims/biometric/formatted_device_info": "iPhone 12 Pro",
        "https://authgear.com/claims/biometric/key_id": "1CC9D95A-6578-4557-8279-C4D5699D3549"
      }
    }
  }
}
```

## Object Reference

The event may contain different objects. You can refer to the below for their attributes.

### The user object

```json5
"payload":{
  "user": {
    "id": "string",
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "last_login_at": "timestamp",
    "is_anonymous": boolean,
    "is_verified": boolean,
    "is_disabled": boolean,
    "is_deactivated": boolean,
    "can_reauthenticate": boolean,
    "standard_attributes": {
      ...
    },
    "custom_attributes": {
      ...
    }
  }
}
```
