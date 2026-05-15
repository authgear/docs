---
description: Overview and examples for the getUser/getUsers queries
---

# Retrieving users using Admin API

The getUser and getUsers queries are a collection of [Admin API](./) queries for getting details about a single user or multiple users using specific attributes as the search key and in real-time.&#x20;

Queries in the collection vary by the type of attribute they support as a search key. The queries are:&#x20;

* `getUsersByStandardAttribute(attributeName: String!, attributeValue: String!)`
* `getUserByLoginID(loginIDKey: String!, loginIDValue: String!)`
* `getUserByOAuth(oauthProviderAlias: String!, oauthProviderUserID: String!)`

### Difference Between Users query and the getUser/getUsers queries

The `users` query is another type of query used to retrieve users. In the following section, we'll discuss the difference between the getUser/getUsers queries and the `users` query.

* **Immediately Consistent:** the getUser and getUsers queries search the database directly for users while the `users` query depends on a search index. As a result, the `users` query may not return details about a recently edited user, while getUser and getUsers queries are immediately consistent.&#x20;
* **Exact Match**: the getUser and getUsers queries only return a result that is an exact match. While calling the `users` query may return users when there is a partial match of a user's email, phone number, or name.
* **Specific attribute**: with the getUser and getUsers queries, you need to specify to retrieve the user by their email, username, or phone number. While the `users` query uses a single `searchKeyword` field.

{% hint style="info" %}
Use the `users` query to search for users without the search term being an exact match. For example, using "John" to search for a user with the full name "John Doe".

Use the getUser and getUsers queries to retrieve the user data after a recent write operation. Note that you can not use any of the getUser and getUsers queries to search for users by their names or any additional attributes outside the supported ones mentioned on this page.
{% endhint %}

The following section contains details and examples for each query in the getUser and getUsers queries:

### 1. getUsersByStandardAttribute

The `getUsersByStandardAttribute` query provides a way to retrieve users using a predefined standard attribute as the search key. The following are the standard attributes (`attributeName`) that you can use:

* `email`
* `preferred_username`
* `phone_number`

You can use the `getUsersByStandardAttribute` query to get details for a user that registered with a `loginID` or OAuth connection. For example, you can use the `email` field in the standard attribute as a search key to find a user who linked an email address from an OAuth connection or registered using the email and password login method.

#### Schema:

```graphql
getUsersByStandardAttribute(
attributeName: String!
attributeValue: String!
): [User!]!
```

**Inputs**:

* `attributeName`: The name of the standard attribute that will be used as the search key. The value can only be: `email`, `preferred_username`, or `phone_number`.
* `attributeValue`: The actual value for the user's standard attribute that you're using (`attributeName`) as the search key. For example, the full email address of the user if you are using `email` as the value for `attributeName`.

#### Example:

{% tabs %}
{% tab title="Query" %}
```graphql
query {
  getUsersByStandardAttribute(attributeName: "email", attributeValue: "user@example.com") {
    id,
    standardAttributes
  }
}
```
{% endtab %}

{% tab title="Response" %}
```json
{
  "data": {
    "getUsersByStandardAttribute": [
      {
        "id": "VXNlciklODRkMzdjZi1hZDQ5LTRiZDItOTMzZJ2tOGY1YThlYjc34RE",
        "standardAttributes": {
          "email": "user@example.com",
          "email_verified": true,
          "family_name": "Aboyi",
          "given_name": "John",
          "name": "John Doe",
          "nickname": "Pius",
          "picture": "https://platform-lookaside.fbsbx.com/.../",
          "updated_at": 1724858514
        }
      }
    ]
  }
}
```
{% endtab %}
{% endtabs %}

### 2. getUserByLoginID

Use the `getUserByLoginID` query to retrieve details about a user using their `loginID` as the search key. The following are the types of `loginID` supported:

* `email`
* `username`
* `phone`

**Note**: An email address linked to a user's account via social/enterprise login (OAuth) provider only can not be used as a search key in the getUserByLoginID query. Use [getUsersByStandardAttribute](retrieving-users-using-admin-api.md#id-3.-getusersbystandardattribute) instead to search for email addresses linked via OAuth provider only.

#### Schema:

<pre class="language-graphql"><code class="lang-graphql">getUserByLoginID(
  loginIDValue: String!
<strong>  loginIDKey: String!
</strong>): User
</code></pre>

**Inputs**:

* `loginIDKey`: the value of this field should be the type of loginID associated with the user's account ( `email`, `phone` or `username`).
* `loginIDValue`: enter the exact value of the identity associated with the user's account. For example, the user's full email address if their identity is `email`. For `phone`, use the full phone number including the "+" sign and country code, e.g. _+441234567890_.

#### Example:

The following example shows a `getUserByLoginID`  query and a response when a user is found.

{% tabs %}
{% tab title="Query" %}
```graphql
query {
  getUserByLoginID(loginIDKey: "email", loginIDValue: "user@example.com") {
    id,
    standardAttributes
  }
}
```
{% endtab %}

{% tab title="Response" %}
```json
{
  "data": {
    "getUserByLoginID": {
      "id": "VXNlciklODRkMzdjZi1hZDQ5LTRiZDItOTMzZJ2tOGY1YThlYjc34RE",
      "standardAttributes": {
        "email": "user@example.com",
        "email_verified": true,
        "family_name": "Doe",
        "given_name": "John",
        "updated_at": 1724854753
      }
    }
  }
}
```
{% endtab %}
{% endtabs %}

### 3. getUserByOAuth

The `getUserByOAuth` query allows you to retrieve details about a user that linked an identity using a social/enterprise provider. For example, you can use the `getUserByOAuth` query to search for a user that linked their account to Facebook using the User ID (sub) issued by Facebook OAuth provider as the search key.

#### Schema:

```graphql
getUserByOAuth(
  oauthProviderAlias: String!
  oauthProviderUserID: String!
): User
```

**Inputs**:

* `oauthProviderAlias`: This is an identifier for each OAuth provider that can be set in Authgear portal via **Authentication** > **Social / Enterprise Login**. The default for value is `google`, `facebook`, `github`, `apple` for respective social login providers.
* `oauthProviderUserID`: The value for this field should be the `sub` (User ID) returned by the OAuth provider. You can see the value of a user's oauthProviderUserID in their `oauthConnections.claims` under the `id` field. See the official documentation for each OAuth provider to learn more about their sub. [This page](https://www.facebook.com/help/1397933243846983?helpref=faq\_content) shows an example of `sub` from Facebook.

#### Example

{% tabs %}
{% tab title="Query" %}
```graphql
query {
  getUserByOAuth(oauthProviderAlias: "facebook", oauthProviderUserID: "1234567812730389") {
    id,
    oauthConnections {
      claims
    }
  }
}
```
{% endtab %}

{% tab title="Response" %}
```json
{
  "data": {
    "getUserByOAuth": {
      "id": "VXNlciklODRkMzdjZi1hZDQ5LTRiZDItOTMzZJ2tOGY1YThlYjc34RE",
      "oauthConnections": [
        {
          "claims": {
            "email": "user@example.com",
            "family_name": "Doe",
            "given_name": "John",
            "https://authgear.com/claims/oauth/profile": {
              "email": "user@example.com",
              "first_name": "John",
              "id": "1234567812730389",
              "last_name": "Doe",
              "name": "John Doe",
              "name_format": "{first} {last}",
              "picture": {
                "data": {
                  "height": 50,
                  "is_silhouette": false,
                  "url": "https://platform-lookaside.fbsbx.com/..../",
                  "width": 50
                }
              },
              "short_name": "John"
            },
            "https://authgear.com/claims/oauth/provider_alias": "facebook",
            "https://authgear.com/claims/oauth/provider_type": "facebook",
            "https://authgear.com/claims/oauth/subject_id": "1234567812730389",
            "name": "John Doe",
            "nickname": "John",
            "picture": "https://platform-lookaside.fbsbx.com/../"
          }
        }
      ]
    }
  }
}
```
{% endtab %}
{% endtabs %}

