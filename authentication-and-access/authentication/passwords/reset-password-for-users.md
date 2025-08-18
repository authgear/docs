---
description: Guide on resetting a user's password via the Authgear Portal or AdminAPI
---

# Reset Password for Users

## How to reset a user's password via Authgear Portal <a href="#via-portal" id="via-portal"></a>

{% stepper %}
{% step %}
In your project, navigate to **User Management** > **Users**. Then click on the user whose password you would like to reset.

<figure><img src="../../../.gitbook/assets/Screenshot 2025-08-05 at 16.06.11.png" alt=""><figcaption></figcaption></figure>


{% endstep %}

{% step %}
Go to the user's **Account Security** tab, and click on Change Password. You can also require the user to change their password on login by enabling the **Ask to change password on login** toggle.&#x20;

<figure><img src="../../../.gitbook/assets/Screenshot 2025-08-05 at 16.06.50.png" alt=""><figcaption></figcaption></figure>


{% endstep %}

{% step %}
You can either enter a secure password you wish to set for the user in the **Password** field, or click on **Generate** to have Authgear automatically create a new password for the user.

Check the "**Send the password to user's email**" box to enable Authgear to send the password entered in the **Password** field to the user.

If you wish to force users to change their password upon logging in with the password you set for them, enable "Ask user to change password on login". Otherwise, users can continue using the password you set for them.

<figure><img src="../../../.gitbook/assets/Screenshot 2025-08-05 at 16.15.42.png" alt=""><figcaption></figcaption></figure>

Finally, remember to click **Change** to save your changes.&#x20;
{% endstep %}
{% endstepper %}

## How to reset a user's password via AdminAPI <a href="#via-portal" id="via-portal"></a>

See [resetPassword](../../../api-reference/apis/admin-api/api-queries-and-mutations.md#id-2.9.-resetpassword) under Reference > APIs > Admin API > API Queries and Mutations.

## What happens if the user loses the preset password?

Users can still log in to their new account if they lose the password you set for them. They can click **Forgot Password** during login, and an OTP or link will be sent to the user. The user can then set a new password.
