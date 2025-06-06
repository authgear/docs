# SAML Attribute Mapping

By default, only the `sub` field of the [UserInfo](../../../reference/apis/oauth-2.0-and-openid-connect-oidc/userinfo.md) is included in the SAML assertion. To include other fields like users' email address, phone number, etc., you can set up SAML Attribute Mapping for the fields.

SAML Attribute Mapping allows you to configure your Authgear client application to include additional fields in the SAML assertion. This is a great way to pass additional data from Authgear to your SAML application that depends on Authgear as an Identity Provider.

Authgear supports using SAML attribute mapping to include additional fields in the SAML assertion using any field in the user profile attributes (UserInfo).&#x20;

There's also support for a template that can be used to customize the values of the fields before including them in the SAML assertion. For example, the template `{{.preferred_username}}@example.com` will return the `preferred_username` field from the UserInfo prepended to '@example.com'.

{% hint style="info" %}
To enable SAML Attribute Mapping for your Authgear project, [please contact support](https://www.authgear.com/schedule-demo).
{% endhint %}

### Example of SAML Attribute Mapping

```
attributes:
      definitions:
      - name: family_name
      - name: given_name
      - name: placeholder_email
      mappings:
      - from:
          user_profile:
            pointer: /given_name
        to:
          saml_attribute: given_name
      - from:
          user_profile:
            pointer: /family_name
        to:
          saml_attribute: family_name
      - from:
          text_template:
            template: '{{.preferred_username}}@example.com'
        to:
          saml_attribute: placeholder_email
```

The above example creates three SAML attributes (`family_name`, `given_name`, `placeholder_email`) that will be added to the SAML assertion.

The value for `pointer` refers to a user profile attribute in the UserInfo object.

From an attribute that uses a template, the value in between the `{{}}` also refers to a user profile attribute in the UserInfo object.\


