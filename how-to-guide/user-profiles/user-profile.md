# What is User Profile

The user profiles contain information about your end-users such as name, email, addresses, and their unique identifier. You can manage the profiles via the Portal & Admin API. The end-users can also manage their own profile through the Profile section in the [User Setting page](../built-in-ui/auth-ui.md) provided by the AuthUI.

The complete information in the user profiles is a combination of standard attributes and custom attributes. Attributes are a way of grouping the fields of the user profile information. With standard attributes containing common fields, you'll find in a user profile, hence the names of these fields are set by Authgear. You set custom fields on the other hand based on the unique needs of your project.

## Standard Attributes

The following attributes are built-in supported by Authgear. They are the set of [**Standard Claims** defined by the OIDC specifications](https://openid.net/specs/openid-connect-core-1\_0.html#StandardClaims). Some of them are default hidden from the Admin Portal and end-users. Their visibility and mutability can be configured through the Admin Portal.

| Attribute name | Default Visibility | Format                                                                                     |
| -------------- | ------------------ | ------------------------------------------------------------------------------------------ |
| Name           | Hidden             | String                                                                                     |
| Given Name     | Editable           | String                                                                                     |
| Family Name    | Editable           | String                                                                                     |
| Middle Name    | Hidden             | String                                                                                     |
| Nickname       | Hidden             | String                                                                                     |
| Profile        | Hidden             | URL String                                                                                 |
| Picture        | Editable           | URL String                                                                                 |
| Website        | Hidden             | URL String                                                                                 |
| Gender         | Editable           | `male`, `female` or Custom String                                                          |
| Birthdate      | Editable           | Date in YYYY-MM-DD                                                                         |
| Timezone       | Editable           | [tz database zone name](https://en.wikipedia.org/wiki/List\_of\_tz\_database\_time\_zones) |
| Language       | Editable           | BCP47 language tag enabled by the project                                                  |
| Address        | Hidden             | JSON Object                                                                                |

### Standard Attributes that are coupled with Identities

The following attributes are coupled with the [identities](../../concepts/user-identity-and-authenticator.md#identity) owned by the end-user. They represent the email addresses, phone numbers, or usernames the end-users are using to authenticate themselves on Authgear. If the end-user uses a third-party identity provider for authentication, these attributes will be coupled with the corresponding attributes returned by the provider.

The standard attributes coupled with identities are listed below:

* `email`
* `email_verified`
* `phone_number`
* `phone_number_verified`
* `preferred_username`

The above attributes are coupled with identities when a user specifies them during sign-up, when users verify their email address or phone number, or from **User Management** > **Users** > select a user > **Identities** in Authgear portal or via the Admin API.

The Login Methods enabled for an Authgear project affect the identities available.

## Custom Attributes

You can define a set of custom attributes in the user profile. They are returned as a JSON object in under the `custom_attributes` key in `userInfo`.

```json5
{
    ...,
    "custom_attributes": {
        "department": "example department"
    }
}
```

### Add new custom attributes

Go to **Portal** > **User Profile** > **Custom Attributes** and click **Add New Attribute**

The custom attribute name should consist of lowercase letters (a-z), digits (0-9) and underscore (\_) only. It must start with lowercase letters (a-z), and NOT end with an underscore (_\__). The default display name will be the attribute name split with underscore and in title case. e.g. `my_string` will render as `My String` in the [AuthUI Settings page](../built-in-ui/auth-ui.md).

Authgear supports the following attribute types:

* String
* Number
* Integer
* Dropdown
* Phone Number
* Email Address
* URL
* Country Code

### Modify custom attributes

You can change a custom attribute name and validation settings such as min and max value. The attribute type cannot be changed once it's set. To migrate the attribute into a new type, create a new attribute, migrate the values from the old to the new one, and then change the name and access right of the old attribute to make it obsolete.

### Delete custom attributes

Deleting custom attributes is not supported. You can change the name and access rights to make an attribute obsolete.

### Custom attribute order

You can arrange the attribute order by drag-and-drop the handle in the custom attribute configuration in the Portal. This will control the order of how the attributes are shown to the end-users in the [AuthUI User Settings page](../built-in-ui/auth-ui.md).

## User Profile Configuration

The access rights for different parties on individual attributes can be configured through the Authgear Portal. Under the hood, all the attributes are available, however, they can be configured to be `hidden` or `read-only` according to the needs of your projects to avoid confusion.

These are the parties that have access to the user profile:

### The Admin API

Through [the Admin API](../../reference/apis/admin-api/), developers **ALWAYS** have **full access** to **ALL** the standard attributes and custom attributes. The Admin API allows the developer to view or edit the standard attributes and the custom attributes.

### The Portal

The admin user can view or edit the standard attributes via the Authgear Portal.

### The Session Bearer

The session bearer is someone who has a valid session cookie or a valid access token. The standard attributes of the end-user whom the session represents can be viewed by accessing [the UserInfo endpoint](user-profile.md#userinfo-endpoint) and [the resolver endpoint](https://docs.authgear.com/get-started/backend-integration/nginx). The session bearer can be the end-user, the client mobile app, or the client website.

### The End-user

The end-user can view or edit the standard attributes through the Profile section in the [User Setting page](../built-in-ui/auth-ui.md) provided by the AuthUI.

## Profiles from Third-party Identity Providers

Authgear supports various [social and enterprise identity providers](../how-to-setup-sso-integrations/). End-users can sign up and log in to your apps via these connections. Upon signup, these providers will return a set of user attributes about the end-user. Authgear will copy those attributes and populate the profile of the end-user.

More info about the population logic can be found in [the specification](https://github.com/authgear/authgear-server/blob/master/docs/specs/user-profile/design.md#standard-attributes-population).
