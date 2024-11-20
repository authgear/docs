# Search for users

Authgear Admin GraphQL API supports searching user by keywords. Keywords include **email**, **phone number**, **username**, **SSO subject id** and **standard attributes**.

## Search users by email

You can use the user search query to search user by email and obtains the user details.

In the following example, the project wants to search the user by email and check if they haven't set up the TOTP authenticator.

You can adjust the fields in the query based on your needs. To explore more supporting fields, you can [try it out via the GraphiQL tool](../#trying-out-the-admin-api-graphql-endpoint).

The query has a maximum limit of 20, pagination parameters should be provided to obtain all the results. See [detail](search-for-users.md#pagination).

### The query

```graphql
query {
  users(
    searchKeyword: "user@example.com"
  ) {
    edges {
      node {
        id
        createdAt
        lastLoginAt
        isAnonymous
        isDisabled
        disableReason
        isDeactivated
        deleteAt
        standardAttributes
        loginIDs {
          id
          claims
        }
        secondaryTOTPAuthenticators {
          id
          claims
        }
      }
    }
    totalCount
  }
}
```

### The sample response

```json
{
  "data": {
    "users": {
      "edges": [
        {
          "node": {
            "id": "VXNlcjphZTNjMGFiZS02YjdkLTQ1ZjktOWIzZS1jMDUwYjVmY2Q3NjI",
            "lastLoginAt": "2006-01-02T03:04:05.123456Z",
            "createdAt": "2006-01-02T03:04:05.123456Z",
            "deleteAt": null,
            "isDisabled": false,
            "disableReason": null,
            "isAnonymous": false,
            "isDeactivated": false,
            "standardAttributes": {
              "email": "user@example.com",
              "email_verified": true,
              "name": "user01",
              "updated_at": 1136171045
            },
            "loginIDs": [
              {
                "claims": {
                  "email": "user@example.com",
                  "https://authgear.com/claims/login_id/key": "email",
                  "https://authgear.com/claims/login_id/original_value": "user@example.com",
                  "https://authgear.com/claims/login_id/type": "email",
                  "https://authgear.com/claims/login_id/value": "user@example.com"
                },
                "id": "SWRlbnRpdHk6NDNhMzZhMTEtM2VkNS00YjczLWE0ZjktMjQ1MWYyMzM5MmVj"
              }
            ],
            "secondaryTOTPAuthenticators": [
              {
                "claims": {
                  "https://authgear.com/claims/totp/display_name": "TOTP @ 2006-01-02T03:04:05Z"
                },
                "id": "QXV0aGVudGljYXRvcjo3NGY0NWUzNi0xMGEyLTQ0MjctYjMxYS0yY2Q3NjBjZDU4MTc"
              }
            ]
          }
        }
      ],
      "totalCount": 1
    }
  }
}
```

## Pagination

### The query

```graphql
query {
  users(
    first: 5
    after: "b2Zmc2V0OjQ"
    searchKeyword: ""
    sortBy: CREATED_AT
    sortDirection: DESC
  ) {
    edges {
      node {
        id
      }
      cursor
    }
    totalCount
  }
}
```

### The sample response

```json
{
  "data": {
    "users": {
      "edges": [
        {
          "cursor": "b2Zmc2V0OjU",
          "node": {
            "id": "VXNlcjo2YTExYjkxNy0yNTg0LTRmNWEtOTkyMS0xN2E3ZmMzMWZjZWU"
          }
        },
        {
          "cursor": "b2Zmc2V0OjY",
          "node": {
            "id": "VXNlcjo0ZTFhZjZmYy0zM2VjLTQ5NjAtOGI2ZC00YTc5YjkxM2Q5N2Y"
          }
        },
        {
          "cursor": "b2Zmc2V0Ojc",
          "node": {
            "id": "VXNlcjphZDJjZWY5Ny0xYzk2LTQ4N2ItOWQzZS01NGVjMzJhYzUxYjY"
          }
        },
        {
          "cursor": "b2Zmc2V0Ojg",
          "node": {
            "id": "VXNlcjphMWIzMzVhMC1jYTdiLTRjMTItYmVmNC04NmNiZTQ4OGMxNzI"
          }
        },
        {
          "cursor": "b2Zmc2V0Ojk",
          "node": {
            "id": "VXNlcjphOWU2YTYwYy0wYTQ1LTQxMzctYTM3MC0wNWM1MjBlZWFkZmU"
          }
        }
      ],
      "totalCount": 30
    }
  }
}
```

#### Parameters

* `searchKeyword`: Search for users by specified keyword.
* `first`: The number of items to be returned. Maximum is `20`.
* `after`: A cursor for use in pagination. You can pass the cursor value of the last edges item to a subsequent call to fetch the next page of results.
* `sortBy`: The field in which to order users. Supported values: `CREATED_AT` or `LAST_LOGIN_AT`.
* `sortDirection`: The direction in which to order users by the specified field. Supported values: `ASC` or `DESC`.
