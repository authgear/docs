---
description: >-
  API Reference for the  User Import API that developers can use to bulk import
  users from another system to their Authgear project.
---

# User Import API

The User Import API is an API that supports the bulk import of users from another system to an Authgear project. This API is not part of the Admin API GraphQL. However, the API endpoints require the [Admin API JWT token](https://docs.authgear.com/reference/apis/admin-api/authentication-and-security) to access it.

The following are other **important things to note about the User Import API**:

* The actual process of importing the users is asynchronous. This means execution is done in the background. The API provides an endpoint developers can use to query the status of the import.
* Once an import is initiated successfully, the API will return an ID for the task. This ID is required to query the status of the import.
* The body of HTTP requests to the API has a limit of 500KB.
* Using the User Import API does not trigger the `user.pre_create` and `user.created` hooks.
* The API supports the Bcrypt password format. To import passwords using this format specify the format `type` and `password_hash` in an object that will be the value of the user's `password` field. For example:

```json
"password": {
        "type": "bcrypt",
        "password_hash": "$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy"
      },
```

## Endpoints <a href="#endpoints" id="endpoints"></a>

The Import User API has two endpoints, one for initiating a user import task and the other for checking the status of the task. The endpoints only support secure HTTPS request and require a valid Admin API JWT token using `Bearer` authorization header (`Authorization: Bearer <Admin API JWT Token>`).

[![Run In Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/25586250-e38e5bf3-3b2e-4a6e-87d6-c5939e9dadd1?action=collection%2Ffork\&source=rip_markdown\&collection-url=entityId%3D25586250-e38e5bf3-3b2e-4a6e-87d6-c5939e9dadd1%26entityType%3Dcollection%26workspaceId%3D0e6e6700-48e7-498b-a5ed-c9fd5cdd1cf2)

Here are more details about the endpoints and their expected inputs.

### **Initiate Import**

<mark style="color:green;">`POST`</mark> `/_api/admin/users/import`

Use this endpoint to create a new user import task.

**Headers**

| Name          | Value                            |
| ------------- | -------------------------------- |
| Content-Type  | `application/json`               |
| Authorization | `Bearer <Admin API JWT Token>`   |
| Host          | `<Your Authgear Project Domain>` |

**Body**

The Initiate Import endpoint accepts JSON input via an HTTP(S) request body. The following is an example of the input:

```json
{
    "identifier": "email",
    "records": [
        {
            "email": "user@example.com",
            "email_verified": true,
            "password": {
                "type": "bcrypt",
                "password_hash": "$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy"
            }
        }
    ]
}
```

#### Input Format <a href="#input-format" id="input-format"></a>

The [endpoint that initiates an import](https://docs.authgear.com/how-to-guide/user-management/import-users-using-user-import-api#initiate-import) accepts JSON input via an HTTP request body. The following sample JSON document shows the expected structure and fields of the input:

```json
{
  "upsert": true,
  "identifier": "email",
  "records": [
    {
      "preferred_username": "jdoe",
      "email": "johndoe@example.com",
      "phone_number": "+85123456789",

      "email_verified": true,
      "phone_number_verified": true,

      "name": "John Doe",
      "given_name": "John",
      "family_name": "Doe",
      "middle_name": "",
      "nickname": "JD",
      "profile": "https://example.com",
      "picture": "https://example.com",
      "website": "https://example.com",
      "gender": "male",
      "birthdate": "1990-01-01",
      "zoneinfo": "Asia/Hong_Kong",
      "locale": "zh-Hant-HK",
      "address": {
        "formatted": "1 Unnamed Road, Central, Hong Kong Island, HK",
        "street_address": "1 Unnamed Road",
        "locality": "Central",
        "region": "Hong Kong",
        "postal_code": "N/A",
        "country": "HK"
      },

      "custom_attributes": {
        "member_id": "123456789"
      },

      "roles": ["role_a", "role_b"],
      "groups": ["group_a"],

      "disabled": false,

      "password": {
        "type": "bcrypt",
        "password_hash": "$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy"
      },

      "mfa": {
        "email": "johndoe@example.com",
        "phone_number": "+85123456789",
        "password": {
          "type": "bcrypt",
          "password_hash": "$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy"
        },
        "totp": {
          "secret": "secret"
        }
      }
    }
  ]
}
```

To understand the input better, let's take a close look at the three fields (`upsert`, `identifier`, `records`) that are directly on the root of the above JSON document.

* `upsert`: This is an optional boolean that is `false` by default. When the value for this field is set to `true` and a user already exists with the same identity, the user's data is updated based on the [update behavior](https://docs.authgear.com/how-to-guide/user-management/import-users-using-user-import-api#update-behavior) for each attribute.
* `identifier`: This field is required. It tells Authgear what attribute to use to identify an existing user. The following strings are the accepted values: `preferred_username`, `email`, and `phone_number`.
* `records`: This is where a developer can provide the data of all the users they wish to import in an array. Each direct object in the array represents a single user. Within the object, you can define the standard attributes for the user using the various fields as shown in the sample above. You may also define custom attributes in an object nested inside the `custom_attributes` field as also shown above.

**Response**

{% tabs %}
{% tab title="200" %}
```json
{
  "id": "task_4WZ0V7EPT4GZ2ABVN03QXYZ122W835C1",
  "created_at": "2024-04-04T06:56:36.02508096Z",
  "status": "pending"
}
```
{% endtab %}

{% tab title="400" %}
```json
{
  "error": "Invalid request"
}
```
{% endtab %}
{% endtabs %}

`id`: this is the unique ID for the import task that was created. The ID is required to check the status of the task.

`status`: the value for status is `pending` after the import task is created and `completed` once the user import task is finished.

### Check Status

<mark style="color:green;">`GET`</mark> `/_api/admin/users/import/{ID}`

Use this endpoint to query the status of an existing user import task. Replace `{ID}` with a valid ID for a user import task.

**Headers**

| Name          | Value                            |
| ------------- | -------------------------------- |
| Authorization | `Bearer <Admin API JWT Token>`   |
| Host          | `<Your Authgear Project Domain>` |

**Response**

{% tabs %}
{% tab title="200" %}
```json
{
  "id": "task_4WZ0V7EPT4GZ2ABVN03QXYZ122W835C1",
  "created_at": "2024-04-04T06:56:36.02508096Z",
  "status": "completed",
  "summary": {
    "total": 2,
    "inserted": 2,
    "updated": 0,
    "skipped": 0,
    "failed": 0
  },
  "details": [
    {
      "index": 0,
      "record": {
        "email": "user1@example.com",
        "email_verified": true,
        "password": {
          "password_hash": "REDACTED",
          "type": "bcrypt"
        }
      },
      "outcome": "inserted",
      "user_id": "0f0f65ee-4c7d-45a0-a740-bcbbfd3fcf06"
    },
    {
      "index": 1,
      "record": {
        "email": "user2@example.com",
        "email_verified": false,
        "family_name": "Doe",
        "given_name": "John",
        "name": "John Doe",
        "password": {
          "password_hash": "REDACTED",
          "type": "bcrypt"
        }
      },
      "outcome": "inserted",
      "user_id": "9c71fc29-6db6-4a18-aa73-774139fed16d",
      "warnings": [
        {
          "message": "email_verified = false has no effect in insert."
        }
      ]
    }
  ]
}
```
{% endtab %}

{% tab title="400" %}
```json
{
  "error": "Invalid request"
}
```
{% endtab %}
{% endtabs %}

## Update Behavior <a href="#update-behavior" id="update-behavior"></a>

The update behavior for an attribute determines how Authgear will treat that attribute when an existing user has the same value for the specified `identifier` type. For example, if the `identifier` is "email", the update behavior for each attribute is how Authgear will treat the attribute if a user already exists with the same email address as the current user you're trying to import.

Each attribute can have one of the three different types of update behavior described below:

* **UPDATED\_IF\_PRESENT\_AND\_REMOVED\_IF\_NULL**: An attribute with this update behavior will update the user's attribute to the new value if that new value is **not null**. If the new value is explicitly **null**, the attribute will be deleted for the user. And if the attribute is absent, no operation is done.
* **UPDATED\_IF\_PRESENT**: When this is the update behavior of an attribute, it will be updated if it is present. If the attribute is not present, no operation is done.
* **IGNORED**: If a user exists already, the new value of this attribute is ignored. If the attribute is absent, nothing is done.

### **Update Behavior of each field**

The following table shows all attributes and their update behavior for reference purposes:

| Item                                          | Update Behavior                                  | Description                                                                                                                                                                                                            |
| --------------------------------------------- | ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `preferred_username`, `email`, `phone_number` | **UPDATED\_IF\_PRESENT\_AND\_REMOVED\_IF\_NULL** | If it is not `identifier`, then the update behavior applies. The corresponding Login ID will be created, updated or removed as needed.                                                                                 |
| `email_verified`, `phone_number_verified`     | **UPDATED\_IF\_PRESENT**                         | For example, in the first import, if `email_verified` is absent, then email is marked as unverified.                                                                                                                   |
| All other standard attributes                 | **UPDATED\_IF\_PRESENT\_AND\_REMOVED\_IF\_NULL** | In particular, `address` IS NOT merged with the existing value, but REPLACES the existing `address` value.                                                                                                             |
| `custom_attributes.*`                         | **UPDATED\_IF\_PRESENT\_AND\_REMOVED\_IF\_NULL** | For each attribute in `custom_attributes`, the update behavior applies individually. So an absent custom attribute in an upsert does not change the existing value.                                                    |
| `roles`, `groups`                             | **UPDATED\_IF\_PRESENT**                         | If present, the roles and groups of the user will match the value. For example, supposed the user originally has `["role_a", "role_b"]`. `roles` is `["role_a", "role_c"]`. `role_b` is removed and `role_c` is added. |
| `disabled`                                    | **UPDATED\_IF\_PRESENT**                         | Re-importing a record without specifying `disabled` WILL NOT accidentally alter the disabled state previously set by other means.                                                                                      |
| `password`                                    | **IGNORED**                                      | If it was not provided when the record was first imported, subsequent import CANNOT add it back.                                                                                                                       |
| `mfa.email`                                   | **UPDATED\_IF\_PRESENT\_AND\_REMOVED\_IF\_NULL** | If provided, the user can perform 2FA with email OTP.                                                                                                                                                                  |
| `mfa.phone_number`                            | **UPDATED\_IF\_PRESENT\_AND\_REMOVED\_IF\_NULL** | If provided, the user can perform 2FA with phone OTP.                                                                                                                                                                  |
| `mfa.password`                                | **IGNORED**                                      | If it was not provided when the record was first imported, subsequent import CANNOT add it back.                                                                                                                       |
| `mfa.totp`                                    | **IGNORED**                                      | If it was not provided when the record was first imported, subsequent import CANNOT add it back.                                                                                                                       |

## Usage Example

The following is a detailed guide with examples of using the User Import API to bulk import users and check the status of tasks.

{% content-ref url="../../admin/user-management/import-users-using-user-import-api.md" %}
[import-users-using-user-import-api.md](../../admin/user-management/import-users-using-user-import-api.md)
{% endcontent-ref %}
