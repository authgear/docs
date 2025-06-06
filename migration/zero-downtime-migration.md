# Zero-downtime migration

The zero-downtime migration approach is much like bulk migration but without users that are already logged-in via the old authentication system having to log in again. This is made possible by taking the access token from the old system, verifying it, and then exchanging that access token for a new access token from Authgear.

To exchange a valid access token from your old system, implement a migration endpoint that calls the **CreateSession** mutation of Authgear's Admin API.

<figure><img src="../../.gitbook/assets/authgear-zero-dt-migration.png" alt=""><figcaption><p>flow of zero-downtime migration</p></figcaption></figure>

### When to use Zero-downtime Migration

Zero-downtime migration is ideal in the following scenarios:

* When it is important to keep users logged in after migration without making them repeat the login process.
* In an application with a large number of logged-in users that are not usually required to login every time they return to your application.

**Pros**

Your users are not logged out. Hence, there is no need for them to repeat login.

**Cons**

It is more complex to set up and deploy.

{% hint style="info" %}
Note: For security reasons, the CreateSession mutation is disabled by default. Hence, you need to [contact us](https://www.authgear.com/schedule-demo) to enable it for your project.&#x20;
{% endhint %}

In this guide, you'll learn how to perform a zero-downtime migration such that users who are currently logged in using your old authentication system are seamlessly moved to Authgear without being logged out at any point during the migration.

For this guide, we'll be referring to the source code for a demo Express.js app. The app will take a verified access token from your existing authentication provider details for the user associated with that access token to create a new Authgear user session (access token).&#x20;

### Step 1: Disable Update to User Data on Old Authentication System

First, temporarily restrict your users from changing their passwords and other profile information on your legacy system to prevent inconsistency in data during migration.

The actual operations required to do this will vary depending on your old authentication provider. You should refer to their official documentation for more details.

### Step 2: Import User Data to Authgear

Import the user data you have already exported from your legacy system into Authgear using the [User Import API](../../reference/apis/user-import-api.md).

Your user data must be reformatted using the [User Import API Schema](../../reference/apis/user-import-api.md#input-format).

Once you have formatted your user data, send the JSON data in an HTTPS request to the `_api/admin/users/import` endpoint of the User Import API.

The step for importing users in zero-downtime migration is the same as in [bulk migration](bulk-migration.md).

### Step 3: Implement and Deploy Updated Middleware and Server-side Code

Implement middleware and server-side code that will recognize sessions from both Authgear and your old authentication provider.

Your implementation should be able to send an access token and a delegated identifier (login ID) from your old system to the migration endpoint that we will implement later in this guide.

```javascript
const retrieveSessionFromOldSystem = () => {
    // ... implement the logic for retrieving session from old system here.
    // also you can return the user's attribute from the old system that was used as identifier in your import.
    // for example, if "identifier": "email", return the user's email. this will be used as loginID to retrieve the user's ID from Authgear

    return {
        loginID: "user@example.com",
        accessToken: "ey..."
    };
}
```

### &#x20;Step 4: Implement Migration Endpoint

Now, implement an endpoint in your application that will verify the access token from your old system, and if the access token is valid, proceed.

Our migration endpoint for the demo Express.js app in this guide uses the user's email address as a login ID. This is because we are assuming we have set `email` as the identifier. This identifier is set during the bulk import of user data to Authgear. If you set `phone` as the identifier, you can use the user's phone number as login ID instead.

{% hint style="info" %}
For the rest of this step, we will be making HTTPS requests to the Admin API GraphQL endpoint. A valid Admin API JWT is required for this. See [Authenticate Admin API](../../reference/apis/admin-api/authentication-and-security.md) to learn more.
{% endhint %}

After you have verified the access token, use the user's login ID to call [the Admin API](../../reference/apis/admin-api/)'s `getUserByLoginID` query.

```javascript
const admin_api_endpoint = appUrl + "/_api/admin/graphql";
    const getUserQuery = await axios.post(
        admin_api_endpoint,
        {
          query: `query {
            getUserByLoginID(loginIDKey: "email", loginIDValue: "user@example.com")
            {
              id
            }
          }`,
        },
        {
          headers: {
            'Content-Type': 'application/json',
            Authorization: "Bearer " + adminJWT,

          }  
        }
      );

      const userID = getUserQuery.data.data.getUserByLoginID.id;

```

Specify your identifier type and login ID using `loginIDKey` and `loginIDValue` respectively in the input for the `getUserByLoginID` query.&#x20;

The `getUserByLoginID` query will return a base64 encoded version of the current user's `sub` (user ID) in `data.getUserByLoginID.id`. You will use this user ID in the `createSession` mutation.

With a valid base64 encoded user ID, call the createSession mutation to generate a new Authgear session (with access token and refresh token).

```javascript
//now create a new session
const createSessionMutation = await axios.post(
  admin_api_endpoint,
  {
    query: `mutation {
      createSession(input: {userID: "${userID}", clientID: "<YOUR_AUTHGEAR_CLIENT_ID>"})
      {
        accessToken,
        expiresIn,
        refreshToken
      }
    }`,
  },
  {
    headers: {
      'Content-Type': 'application/json',
      Authorization: "Bearer " + adminJWT,

    }  
  }
);

const authgearAccessToken = createSessionMutation.data.data.createSession.accessToken;
```

You can now use the Authgear access token to protect resources on your application. All users will continue to use your updated app without any interruption, as the migration endpoint will generate new Authgear sessions without prompting your users to log in again. &#x20;
