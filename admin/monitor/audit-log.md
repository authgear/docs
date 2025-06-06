# Audit Log For Users Activities

Authgear provides the event logs for you to analyze security issues and monitor the business.

## View and retrieve logs

You can view the audit log in the Portal, or retrieve logs using the [Admin API](../../api-reference/apis/admin-api/).

### View in Portal

The portal provides an interface for you to look up the log by event and date range.

The **Users Activities** tab on the Audit Log page filters the log to only show activities performed by a normal user or by an admin on a user's profile.

<figure><img src="../../.gitbook/assets/audit-user-tab (1).png" alt=""><figcaption></figcaption></figure>

### Retrieve with Admin API

The API schema can be found in the [Admin API GraphiQL Explorer](../../api-reference/apis/admin-api/#api-explorer). For example:

```graphql
query {
  auditLogs(first:5){
    edges{
      node{
        activityType
        clientID
        createdAt
        data
      }
    }
  }
}
```

#### Filtering Audit Log

When using the Admin API, you can filter the Audit Log by an attribute like **activityTypes** to omit records you're not interested in.

The following is an example that includes filters.

```graphql
query {
  auditLogs(first:5, activityTypes:[USER_AUTHENTICATED,USER_DELETED]){
    edges{
      node{
        activityType
        clientID
        createdAt
        data
      }
    }
  }
}
```

The above query will only return events with activity types `USER_AUTHENTICATED` and `USER_DELETED`.

## Log events

Here is the list of activity types that are logged:

#### Authentication failed

| Activity type                                      | Description                                                                      |
| -------------------------------------------------- | -------------------------------------------------------------------------------- |
| AUTHENTICATION\_IDENTITY\_ANONYMOUS\_FAILED        | Anonymous user authentication failed                                             |
| AUTHENTICATION\_IDENTITY\_BIOMETRIC\_FAILED        | Authentication with biometric failed                                             |
| AUTHENTICATION\_IDENTITY\_LOGIN\_ID\_FAILED        | A user's login attempt failed because the email or user ID provided is not found |
| AUTHENTICATION\_PRIMARY\_OOB\_OTP\_EMAIL\_FAILED   | Authentication using the OTP sent via email failed                               |
| AUTHENTICATION\_PRIMARY\_OOB\_OTP\_SMS\_FAILED     | Authentication using the OTP sent via SMS failed                                 |
| AUTHENTICATION\_PRIMARY\_PASSWORD\_FAILED          | A user entered an invalid password during a login attempt                        |
| AUTHENTICATION\_SECONDARY\_OOB\_OTP\_EMAIL\_FAILED | 2FA via email failed                                                             |
| AUTHENTICATION\_SECONDARY\_OOB\_OTP\_SMS\_FAILED   | 2FA via SMS failed                                                               |
| AUTHENTICATION\_SECONDARY\_PASSWORD\_FAILED        | Secondary authentication using password failed                                   |
| AUTHENTICATION\_SECONDARY\_RECOVERY\_CODE\_FAILED  | Recovery code verification failed                                                |
| AUTHENTICATION\_SECONDARY\_TOTP\_FAILED            | A 2FA attempt failed                                                             |

#### Identity changes

| Activity type                 | Description                                                  |
| ----------------------------- | ------------------------------------------------------------ |
| IDENTITY\_BIOMETRIC\_DISABLED | Biometric login is disabled by a user or an admin            |
| IDENTITY\_BIOMETRIC\_ENABLED  | User enabled biometric login                                 |
| IDENTITY\_EMAIL\_ADDED        | A user or admin added a new email to an existing user        |
| IDENTITY\_EMAIL\_REMOVED      | An email address was removed from an existing user's profile |
| IDENTITY\_EMAIL\_UPDATED      | A user updated their email address                           |
| IDENTITY\_OAUTH\_CONNECTED    | A profile is linked to OAuth                                 |
| IDENTITY\_OAUTH\_DISCONNECTED | A profile is unlinked from OAuth                             |
| IDENTITY\_PHONE\_ADDED        | A user or admin added a new phone number to an existing user |
| IDENTITY\_PHONE\_REMOVED      | A phone number was removed from an existing user's profile   |
| IDENTITY\_PHONE\_UPDATED      | A user updated their phone number                            |
| IDENTITY\_USERNAME\_ADDED     | A user or admin added a new username to an existing user     |
| IDENTITY\_USERNAME\_REMOVED   | A user or admin removed the username for a user              |
| IDENTITY\_USERNAME\_UPDATED   | The username for a user was updated                          |

#### User actions

| Activity type               | Description                                                                      |
| --------------------------- | -------------------------------------------------------------------------------- |
| USER\_ANONYMOUS\_PROMOTED   | This event is triggered when an anonymous user is promoted to a normal user      |
| USER\_AUTHENTICATED         | Successful user sign-in                                                          |
| USER\_CREATED               | A new user successfully registered                                               |
| USER\_DELETED               | A user account is deleted                                                        |
| USER\_DELETION\_SCHEDULED   | A user account deletion is scheduled                                             |
| USER\_DELETION\_UNSCHEDULED | A previously scheduled user deletion is unscheduled                              |
| USER\_DISABLED              | User account disabled. An admin disabling a users account can trigger this event |
| USER\_PROFILE\_UPDATED      | A user updated details like their profile name, gender and more                  |
| USER\_REENABLED             | A user account that was previously disabled is enabled                           |
| USER\_SESSION\_TERMINATED   | An active user session is terminated                                             |
| USER\_SIGNED\_OUT           | A user that was signed in logged out                                             |

#### Others

| Activity type           | Description                                                         |
| ----------------------- | ------------------------------------------------------------------- |
| WHATSAPP\_OTP\_VERIFIED | User completed a verification process using WhatsApp to receive OTP |
| SMS\_SENT               | An SMS notification like OTP was sent to a user                     |
| EMAIL\_SENT             | An email notice like verification code was sent to a user           |

## Log data

Each audit log event contains the following attributes in their data

<table><thead><tr><th width="278">Attribute</th><th>Description</th></tr></thead><tbody><tr><td><code>id</code></td><td>Unique identifier of the event</td></tr><tr><td><code>seq</code></td><td>Sequence number of the event</td></tr><tr><td><code>type</code></td><td>Activity type</td></tr><tr><td><code>context</code></td><td>The who, when and where of the event triggered. e.g. IP address, user agent, user ID, timestamp</td></tr><tr><td><code>payload</code></td><td><p>Relevant data according to the event type:</p><p><strong>Messaging (SMS, Email OTP):</strong> the phone number/email address of the receiver</p><p><strong>Authentication/Identity/User actions:</strong> a snapshot of the related session and user attributes</p></td></tr></tbody></table>
