# Rolling migration

With rolling migration, you can gradually transfer user data from your old system to Authgear. For example, you can move users with certain feature flags, like users from specific regions. Or configure your application such that all new sign-ups use Authgear.

In this guide, you'll learn how to migrate users from your old authentication system to Authgear using the rolling migration approach.

For this guide, we'll be referring to the source code for a demo Express.js app. The demo app is an application that is moving new users sign ups from an old authentication system to Authgear.

### Step 1: Deploy Middleware to Support Authgear Session and Old Authentication System

The first step for setting up rolling migration is to implement a middleware or server-side logic that supports both Authgear session and your old authentication provider.&#x20;

To do this, implement code to verify both access tokens (sessions) from your old authentication system and Authgear:

```javascript
const axios = require("axios");
const node_jwt = require("jsonwebtoken");
const jwksClient = require("jwks-rsa");

const appUrl = "<YOUR_AUTHGEAR_ENDPOINT>";
const getJwksUri = async (appUrl) => {
  const config_endpoint = appUrl + "/.well-known/openid-configuration";
  const data = await axios.get(config_endpoint);
  return data.data.jwks_uri;
};

// Verify access token from Authgear session.
const authenticateAuthgearToken = async (req, res, next) => {
  const requestHeader = req.headers;
  if (requestHeader.authorization == undefined) {
    return res.sendStatus(401);
  }
  const authorizationHeader = requestHeader.authorization.split(" ");
  const access_token = authorizationHeader[1];

  const decoded_access_token = node_jwt.decode(access_token, {
    complete: true,
  });
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

// Verify session from old authentication provider.
const authenticateOldProviderToken = (req, res, next) => {
    // ... logic for authenticating session from your old provider
    console.log("using old provider session");
    next();
}

```

### Step 2: Enable Authgear for Portion of your Users

Now that you have your app set up to use both Authgear and your old authentication system, you should decide what portion of your users you want to use Authgear.&#x20;

For our demo app here, we'll be enabling Authgear for all users signing up after the 2nd of March 2025 (2025/03/02). You can use another feature flag, such as the location (region or country) of a user.

To be able to determine when a user signed up (or their region or country) before they log in, you may need to set up some external database with their loginID (username, email address, phone number, etc) and the feature flag (sign-up date, country, etc). For our example, we'll use a simple JavaScript object to create a demo user who registered on the 20th of March, 2025.&#x20;

```javascript
const user = { loginID: 'user1@example.com', signup_date: '2025/03/20' };
```

#### Enable Authgear sessions in middleware

For the demo app, we'll use a simple conditional statement to enable Authgear sessions for users who meet the condition for the feature flag (users who signed up after 2025/03/02).

```javascript
let authenticateToken = authenticateOldProviderToken;
const featureFlagDate = new Date('2025/03/02');

const signupDate = new Date(user.signup_date);
if ( signupDate > featureFlagDate) {
    authenticateToken = authenticateAuthgearToken;
}
```

Then, `authenticateToken` can be used as a middleware to protect pages that should be viewed by authenticated users only.

```javascript
app.get("/protected", authenticateToken, (req, res) => {
  res.json({ message: "This is a protected route"});
});
```

### Step 3: Set up your Application to Signup/Login with Authgear and your Old Provider

Update your application to allow users to sign up and log in using Authgear's authentication flow. It is also important to continue supporting your old authentication provider so that old users can still log in.&#x20;

However, you should disable the signup feature on your old system so that all new user data goes to Authgear.

Now, using the feature flag again, we can determine what authentication flow to show a user:

```javascript
app.get("/login", (req, res) => {
    if ( signupDate > featureFlagDate) {
        // TODO start Authgear Authentication flow
    } else {
        // ... logic for authenticating users with your old provider
    }

    res.send("Login")
});
```

We offer SDKs for the following platforms to integrate Authgear into your app seamlessly:

* [React](https://docs.authgear.com/get-started/single-page-app/react)
* [React Native](https://docs.authgear.com/get-started/native-mobile-app/react-native)
* [JavaScript](https://docs.authgear.com/get-started/single-page-app/website)
* [Vue](https://docs.authgear.com/get-started/single-page-app/vue)
* [Android](https://docs.authgear.com/get-started/native-mobile-app/android)
* [iOS](https://docs.authgear.com/get-started/native-mobile-app/ios)
* [Flutter](https://docs.authgear.com/get-started/native-mobile-app/flutter)
* [Ionic](https://docs.authgear.com/get-started/native-mobile-app/ionic-sdk)

Alternatively, you can manually add Authgear to any app as an [OAuth 2.0 provider](https://docs.authgear.com/get-started/regular-web-app).

### Conclusion

The following points are also worth noting while implementing rolling migration:

* For any users who already signed up using the old system and are moved to Authgear via [User Import API](https://docs.authgear.com/reference/apis/user-import-api), they need to be logged out and will be required to log in using the Authgear authentication flow in order to start using Authgear session. Remember to add your feature flag to these users so your application can select Authgear for them.
* Users who are not feature-flagged will continue to see the legacy authentication system. They are not logged out and can continue using their existing session.
* Disable new signups in the legacy system so that all new signups should go through Authgear.
* For new users who go through the new sign-up flow using Authgear, a record in the legacy system should be in place to handle an edge case where the user tries to log in on another device with an older app version.
