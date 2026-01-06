---
description: >-
  Temporarily or permanently suspend a user’s ability to log in and use the
  system
---

# Disable User

The **Disable User** feature allows administrators to temporarily or permanently suspend a user’s ability to log in and use the system.

When a user is disabled, they cannot authenticate and all active sessions are immediately terminated.

### What Does Disabling a User Do?

When a user is disabled:

* Login attempts are blocked
* Existing sessions are terminated immediately
* The user account and data remain intact
* The user can be re-enabled later if the disable period ends or is removed

Disabling a user is reversible unless the account is removed.

### Disable Period Options

You can choose how long a user is disabled:

* **Disable indefinitely**\
  The user remains disabled until an administrator manually re-enables the account.
* **Disable until a specific date**\
  The user is automatically re-enabled after the specified date and time.

When disabling a user, you can provide an optional **disable reason**. The reason is displayed to the user when they attempt to log in. This can reduce support requests and confusion.

### How to Disable a User

You can disable a user with the Admin API or from the Authgear Portal:

1. Go to **Users**
2. Select the user
3. Open the **Account Status** tab
4. Click **Disable user**
5. Choose the disable period
6. (Optional) Enter a disable reason
7. Confirm the action
