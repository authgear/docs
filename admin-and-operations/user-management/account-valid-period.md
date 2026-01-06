---
description: Use account valid period to manage temporary access
---

# Account Valid Period

The **Account Valid Period** feature allows you to define a time window during which a user account is allowed to log in. Outside this period, the user is prevented from logging in.

This is useful for scenarios involving **temporary access**, such as contractors, interns, or external partners.

### What Is Account Valid Period?

By default, a user account can log in indefinitely unless it is disabled or removed.

With **Account Valid Period**, you can:

* Specify a **start time** and an **end time** for account access
* Automatically block login attempts **outside the configured period**
* Avoid manual enable/disable actions for time-bound access

Once the valid period expires (or before it starts), the user will not be able to log in, and existing sessions are not extended beyond the valid period.

### Relationship with User Disable

Account Valid Period and **Disable User** are independent controls.

* Even within the valid period, an administrator can still disable the user to suspend access temporarily
* A disabled user cannot log in, regardless of the valid period
* Re-enabling the user restores access only if the current time is still within the valid period

### How to Configure Account Valid Period

You can configure the account valid period in the following ways:

#### During Account Creation

* Set the start and end time when creating a user in the Authgear Portal
* Set the valid period when creating a user via the Admin API

#### From the User Detail Page

You can also configure or update the valid period for an existing user:

1. Go to **Users**
2. Select a user
3. Open the **Account Status** tab
4. Click **Set Valid Period**
5. Configure the start time and end time

### Common Use Cases

* Granting temporary access to contractors or consultants
* Providing limited-time access for external partners
* Enforcing access expiration for compliance or security requirements
