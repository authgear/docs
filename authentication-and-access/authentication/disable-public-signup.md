# Disable Public Signup

For many business applications, it's important to restrict who can register for your platform. By disabling public signups in Authgear, you can ensure that new users are only added through your internal processes, such as the Admin API or the Authgear Portal.

### When to Disable Public Signup <a href="#when-to-disable-public-signup" id="when-to-disable-public-signup"></a>

Disable public signup if you want to:

* Control user access and approval for your application.
* Provision users only via admin tools.
* Prevent the general public from self-registering.

### How to Disable Public Signup <a href="#how-to-disable-public-signup" id="how-to-disable-public-signup"></a>

To turn off public registration:

1. Navigate to **Portal** > **Advanced** > **Edit Config**.
2.  Find or add the following configuration in your YAML file:

    ```yaml
    authentication:
      public_signup_disabled: true
    # other configurations...
    ```
3. Save your changes

