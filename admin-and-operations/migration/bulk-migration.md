# Bulk migration

The bulk migration strategy allows you to move all your user data to Authgear at once using the [User Import API](../../reference/apis/user-import-api.md). Bulk migration is ideal when you want to stop using your old authentication system immediately and start using Authgear.

Once you import your users, you'll need to implement a middleware and server-side logic that uses Authgear user sessions to protect resources. As a result, all users will be logged out and will be required to log in again using Authgear's authentication flow.

In this guide, you'll learn how to move your user data from your old authentication provider to Authgear using the bulk migration strategy.

### When to use Bulk Migration

The following is a possible scenario where you could use the bulk migration strategy to move user data to Authgear.

* When you are ready to stop using your old authentication system at once and move to Authgear. In that case, you can disable signups on the old system, export your user data, and proceed with the complete steps for bulk migration.

#### **Pros**

* Bulk migration is the most straightforward approach.
* It allows you to migrate all your users at once and stop using your old system as quickly as possible.

#### Cons

* It requires some downtime while you are moving from the old system to Authgear.

### Step 1: Export User Data from Old System

Export all your user data from the old authentication system you're moving from.

The following example shows rows in the `users` table of an application.

```
| Email              | Email Verified | Name      | Given Name | Family Name | Password Type | Password Hash                                                        |
|--------------------|----------------|-----------|------------|-------------|---------------|----------------------------------------------------------------------|
| user1@example.com  | true           |           |            |             | bcrypt        | $2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy         |
| user2@example.com  | false          | John Doe  | John       | Doe         | bcrypt        | $2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy         |
```

You can export users from any authentication system, including your own custom-built system. Then, reformat the data to match Authgear's Import User API schema.

### Step 2: Reformat User Data into Support JSON Format

Reformat your exported user data to a JSON document that follows the [Import User API schema](../../reference/apis/user-import-api.md).&#x20;

You can then use the reformatted JSON document with the Import API to import your user data.

Here's the example `users` table from earlier reformatted:

```json
[
  {
    "email": "user1@example.com",
    "email_verified": true,
    "password": {
      "type": "bcrypt",
      "password_hash": "$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy"
    }
  },
  {
    "email": "user2@example.com",
    "email_verified": false,
    "name": "John Doe",
    "given_name": "John",
    "family_name": "Doe",
    "password": {
      "type": "bcrypt",
      "password_hash": "$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy"
    }
  }
]

```

### &#x20;Step 3: Import Formatted User Data

Import your formatted user data you formatted via a POST HTTPS request to the `_api/admin/users/import` endpoint of the User Import API.

```javascript
const data = {
  identifier: "email",
  records: [
    {
      email: "user1@example.com",
      email_verified: true,
      password: {
        type: "bcrypt",
        password_hash:
          "$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy",
      },
    },
    {
      email: "user2@example.com",
      email_verified: false,
      name: "John Doe",
      given_name: "John",
      family_name: "Doe",
      password: {
        type: "bcrypt",
        password_hash:
          "$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy",
      },
    },
  ],
};

const options = {
  method: "POST",
  headers: {
    "Content-type": "application/json",
    Authorization: "Bearer <ADMIN_API_JWT>",
  },
  body: JSON.stringify(data),
};

const appUrl = "<YOUR_AUTHGEAR_ENDPOINT>";

fetch(`${appUrl}/_api/admin/users/import`, options)
  .then((result) => result.json())
  .then((result) => console.log(JSON.stringify(result)));

```

The above JavaScript code imports the data from our example `users` table using the Import User API. The API requires a valid [Admin API JWT](../../reference/apis/admin-api/authentication-and-security.md) for authentication.&#x20;

Learn more about using the Import User API [here](../user-management/import-users-using-user-import-api.md).

### Step 4: Use Authgear Session in Middleware and Server-side Logic

Now that you have successfully imported your users to Authgear, you can start using Authgear session to protect the pages in your application.

To do this, implement a middleware and server-side logic that uses Authgear session.

{% hint style="info" %}
**Note:** Deploying the middleware and server-side logic that uses Authgear session means all users will be logged out and be required to log in again [using Authgear's authentication flow](broken-reference).
{% endhint %}

The following is an example of a simple Express.js application with a middleware that uses Authgear session:

```javascript
const express = require("express");
const axios = require("axios");
const node_jwt = require("jsonwebtoken");
const jwksClient = require("jwks-rsa");

const app = express();
const port = 3000;

const appUrl = "https://authui.authgeartest.online";
const getJwksUri = async (appUrl) => {
  const config_endpoint = appUrl + "/.well-known/openid-configuration";
  const data = await axios.get(config_endpoint);
  return data.data.jwks_uri;
};

const authenticateToken = async (req, res, next) => {
  const requestHeader = req.headers;
  if (requestHeader.authorization == undefined) {
    return res.sendStatus(401);
  }
  const authorizationHeader = requestHeader.authorization.split(" ");
  const access_token = authorizationHeader[1];

  const decoded_access_token = node_jwt.decode(access_token, {
    complete: true,
  });
  if (decoded_access_token == null) {
    return res.sendStatus(401)
  }
  const jwks_uri = await getJwksUri(appUrl);
  const client = jwksClient({
    strictSsl: true,
    jwksUri: jwks_uri,
  });
  const signing_key = await client.getSigningKey(
    decoded_access_token.header.kid
  );

  try {
    const verify = node_jwt.verify(access_token, signing_key.publicKey, {
      algorithms: ["RS256"],
    });
    console.log(JSON.stringify(verify));
  } catch (error) {
    return res.sendStatus(403);
  }

  next();
};

app.get("/", (req, res) => {
  res.send(
    "Hello world, this is an unprotected page, a good place to start an authentication flow."
  );
});

app.get("/protected", authenticateToken, (req, res) => {
  res.json({ message: "This is a protected route" });
});

app.listen(port, () => {
  console.log(`server started on port ${port}`);
});

```

This example app uses the "Bearer \<access token>" to protect the `/protected` route. Learn more about [verifying access tokens and using Authgear in your backend here.](../../get-started/backend-api/jwt.md)&#x20;

### Step 5: Authenticate Your Users with Authgear

Update your app to allow users to log back in using Authgear's authentication flow.

You can use any of our SDKs for the following platforms to integrate Authgear into your app seamlessly:

* [React](../../get-started/single-page-app/react.md)
* [React Native](../../get-started/native-mobile-app/react-native.md)
* [JavaScript](../../get-started/single-page-app/website.md)
* [Vue](../../get-started/single-page-app/vue.md)
* [Android](../../get-started/native-mobile-app/android/)
* [iOS](../../get-started/native-mobile-app/ios.md)
* [Flutter](../../get-started/native-mobile-app/flutter.md)
* [Ionic](../../get-started/native-mobile-app/ionic-sdk.md)

Or manually add Authgear as an [OAuth 2.0 provider](../../get-started/regular-web-app/).

Done! Now, your user can log in to your app and use the same user account from your old authentication provider.
