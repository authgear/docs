# Update user's standard attributes through Admin GraphQL API

## Overview

To update the user's standard attributes:

1. Fetch the user's original standard attributes by query first.
2. Change the attributes payload and update the user's standard attributes by mutation.

## API Details

### Generate the node ID for fetching the user object

Admin GraphQL API allows you to query different type of objects by using the node query with the node id. The node id is an [base64url](https://datatracker.ietf.org/doc/html/rfc4648#section-5) encoded string with format of `<NODE_TYPE>:<ID>`. 

Generate user's node id examples:

{% tabs %}
{% tab title="Go" %}
```go
package main

import (
	"encoding/base64"
	"fmt"
)

func main() {
	rawStdEncoding := base64.StdEncoding.WithPadding(base64.NoPadding)
	nodeID := rawStdEncoding.EncodeToString([]byte("User:97b1c929-842c-415c-a7df-6967efdda160"))
	fmt.Println(nodeID)
}
```
{% endtab %}
{% tab title="Python" %}
```python
import base64

base64.urlsafe_b64encode(b'User:97b1c929-842c-415c-a7df-6967efdda160').replace(b'=', b'')
```
{% endtab %}
{% endtabs %}

### Fetch the user's standard attributes by query

The query:

```graphql
query ($userID: ID!) {
  node(id: $userID) {
    ... on User {
      standardAttributes
    }
  }
}
```

The variables:

```json
{
  "userID": "<BASE64URL_ENCODED_USER_NODE_ID>"
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

Copy the original standard attributes, change the attributes that you want to update.

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
