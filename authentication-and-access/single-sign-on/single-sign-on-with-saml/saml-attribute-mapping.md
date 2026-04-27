# SAML Attribute Mapping

By default, Authgear includes only the `sub` field in the SAML assertion. SAML Attribute Mapping lets you add other UserInfo fields, such as email, phone number, or name, so your SAML service provider receives the user data it needs.

You can map fields directly from the user profile, or transform them with a Go text template before they go into the assertion.

### How attribute mapping works

Each mapping has two parts:

* **Definition**: declares a SAML attribute by name. This becomes the `Name` of the `<Attribute>` element in the assertion.
* **Mapping**: sets the value of that attribute, either from a UserInfo field or from a text template.

### Enable SAML Attribute Mapping

#### Prerequisites

* A client application with **SAML 2.0 support** enabled. See [.](./ "mention") to set one up.

#### Steps

1. Open the Authgear Portal and go to **Advanced > Edit Config**.
2. Find your application under `saml.service_providers` by matching its `client_id`.
3. Add an `attributes` block under that service provider with `definitions` and `mappings`. Use the example below as a starting point.
4. Save the config.

#### Example

Add the `attributes` block under the service provider matching your `client_id`. Leave the existing fields (`acs_urls`, `audience`, `nameid_format`, etc.) untouched, they were set when the SAML application was created.

```yaml
saml:
  service_providers:
  - client_id: YOUR_CLIENT_ID
    # ...existing fields, leave as-is...
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

The `attributes` block adds three attributes: `family_name`, `given_name`, and `placeholder_email` to every SAML assertion issued to this service provider.

### Configuration reference

#### `definitions`

Each entry declares one SAML attribute.

| Field           | Type   | Required | Description                                                                                                                    |
| --------------- | ------ | -------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `name`          | string | Yes      | The attribute name. Used as the `Name` of the `<Attribute>` element in the assertion.                                          |
| `name_format`   | string | No       | Sets the `NameFormat` attribute. See [SAML Core 2.7.3.1](https://docs.oasis-open.org/security/saml/v2.0/saml-core-2.0-os.pdf). |
| `friendly_name` | string | No       | Sets the `FriendlyName` attribute.                                                                                             |

#### `mappings`

Each mapping sets the value of one declared attribute. Two source types are supported.

**Map from a user profile field**

Use a JSON pointer to reference any field in the UserInfo object.

```yaml
- from:
    user_profile:
      pointer: /email
  to:
    saml_attribute: email
```

| Field                       | Description                                                                                      |
| --------------------------- | ------------------------------------------------------------------------------------------------ |
| `from.user_profile.pointer` | JSON pointer to a UserInfo field, e.g. `/sub`, `/email`, `/phone_number`, `/preferred_username`. |
| `to.saml_attribute`         | Name of the target SAML attribute. Must match one declared in `definitions`.                     |

The SAML value type is derived from the JSON type: strings become `xs:string`, numbers become `xs:decimal`, booleans become `xs:boolean`, and arrays produce multiple `<AttributeValue>` elements.

{% hint style="info" %}
A field referenced by `pointer` must exist in the UserInfo. Missing fields produce an attribute with no `<AttributeValue>`.&#x20;
{% endhint %}

**Map from a text template**

Use a [Go text template](https://pkg.go.dev/text/template) to build the value. The template receives the UserInfo object as its context.

```yaml
- from:
    text_template:
      template: '{{.preferred_username}}@example.com'
  to:
    saml_attribute: placeholder_email
```

The output is always rendered as `xs:string`. Missing fields render as empty strings.

{% hint style="warning" %}
Mapping order matters. If two mappings write to the same `saml_attribute`, the later one overrides the earlier one.
{% endhint %}
