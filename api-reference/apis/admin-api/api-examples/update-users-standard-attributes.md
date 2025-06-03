# Update user's standard attributes

## Overview

To update the user's standard attributes:

1. Fetch the user's original standard attributes by query first.
2. Change the attributes payload and update the user's standard attributes by mutation.

## API Details

### Generate the user node id

Follow the document [here](../node-id.md) to generate the user node id.

### Fetch the user's standard attributes by query

The query:

```graphql
query {
  node(id: "<BASE64URL_ENCODED_USER_NODE_ID>") {
    ... on User {
      standardAttributes
    }
  }
}
```

The sample response:

```json
{
  "data": {
    "node": {
      "standardAttributes": {
        "name": "Name",
        "given_name": "Given Name",
        "family_name": "Family Name",
        "middle_name": "Middle Name",
        "nickname": "Nickname",
        "profile": "https://<PROFILE_URL>",
        "picture": "https://<PICTURE_URL>",
        "website": "https://<WEBSITE_URL>",
        "gender": "male",
        "birthdate": "2020-01-01",
        "zoneinfo": "Europe/London",
        "locale": "en",
        "address": {
          "country": "GB",
          "locality": "locality",
          "postal_code": "postal code",
          "region": "region",
          "street_address": "full street address"
        },
        "updated_at": 1649428136
      }
    }
  }
}
```

### Update the user's standard attributes by mutation

{% hint style="warning" %}
Make sure to copy the original standard attributes as **any attributes that are missing in the payload will be deleted**. Next, change the values for attributes that you want to update.
{% endhint %}

The query:

```graphql
mutation ($userID: ID!, $standardAttributes: UserStandardAttributes!) {
  updateUser(input: {userID: $userID, standardAttributes: $standardAttributes}) {
    user {
      id
      standardAttributes
      updatedAt
    }
  }
}
```

The variables:

```json5
{
  "userID": "<BASE64URL_ENCODED_USER_NODE_ID>",
  "standardAttributes": {
    /* Include all the original standard attributes from the previous response */
    /* and change the attributes that you want to update */
    /* The attributes that are missing in the payload will be deleted */
  }
}
```

The sample variables:

```json
{
  "userID": "<BASE64URL_ENCODED_USER_NODE_ID>",
  "standardAttributes": {
    "name": "Name",
    "given_name": "Given Name",
    "family_name": "Family Name",
    "middle_name": "Middle Name",
    "nickname": "Nickname",
    "profile": "https://<PROFILE_URL>",
    "picture": "https://<PICTURE_URL>",
    "website": "https://<WEBSITE_URL>",
    "gender": "male",
    "birthdate": "2020-01-01",
    "zoneinfo": "Europe/London",
    "locale": "en",
    "address": {
      "country": "GB",
      "locality": "locality",
      "postal_code": "postal code",
      "region": "region",
      "street_address": "full street address"
    }
  }
}
```

The sample response:

```json
{
  "data": {
    "updateUser": {
      "user": {
        "id": "VXNlcjo5N2IxYzkyOS04NDJjLTQxNWMtYTdkZi02OTY3ZWZkZGExNjA",
        "standardAttributes": {
          "name": "Name",
          "given_name": "Given Name",
          "family_name": "Family Name",
          "middle_name": "Middle Name",
          "nickname": "Nickname",
          "profile": "https://<PROFILE_URL>",
          "picture": "https://<PICTURE_URL>",
          "website": "https://<WEBSITE_URL>",
          "gender": "male",
          "birthdate": "2020-01-01",
          "zoneinfo": "Europe/London",
          "locale": "en",
          "address": {
            "country": "GB",
            "locality": "locality",
            "postal_code": "postal code",
            "region": "region",
            "street_address": "full street address"
          },
          "updated_at": 1649428136
        },
        "updatedAt": "2022-04-08T14:40:16.410087Z"
      }
    }
  }
}
```

### Updating Custom Attributes

To learn how to update a user's **custom attributes**, see the [update custom attribute example](https://docs.authgear.com/reference/apis/admin-api/api-queries-and-mutations#id-2.20.-updateuser).
