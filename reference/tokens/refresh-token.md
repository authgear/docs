---
description: >-
  Learn about refresh token, how to get a refresh token and how to use it to get
  a new access token.
---

# Refresh Token

In Authgear, a refresh token is a key that our authorization server returns when an OAuth 2.0 client application successfully exchanges an authorization code for an access token. Usually, in OAuth 2.0, access tokens expire after some time (you can configure this time in the Authgear portal). The refresh token exists to allow your application to request a new access token after the previous one has expired. It eliminates the need for your user to repeat the entire login process all over.

In this post, we'll teach you how to get a refresh token and how to use it to request a new access token after the old one has expired.

### Pre-requisite

* This post assumes you already have some experience with using Authgear as an OIDC provider.
* Have an [Authgear account](https://accounts.portal.authgear.com/signup) and an Authgear application.
* Have the configuration (client ID, authorized redirect URI, endpoints, etc) of your Authgear application noted down.

If you are completely new to Authgear, check out one of our [getting started guides](https://docs.authgear.com/get-started/start-building) first.

## How to Get A Refresh Token

There are many ways to implement Authgear as an Open ID Connect (OIDC) provider for your application based on the programming language or framework you're using. This may determine the procedure you follow to get a refresh token. For example, if you're using any of our official SDKs, there are built-in methods and configurations that simplify the process of getting a refresh token and renewing an expired access token.

However, the key thing that all implementations have in common is including the following `scope` parameter to your application authorization URL:

<pre><code><strong>scope=openid+offline_access
</strong></code></pre>

Or just `scope=offline_access`.

Example of Authgear token endpoint response with refresh token included:

```json
{
  "access_token": "Rjr3abcD1234UVWXYZ1lt",
  "expires_in": 1800,
  "id_token": "eyKdsdj88sdjsdjjdfjjdfjdfjdfjkskslslslmd.eyshdhsdhnsjdksdjd7783jdjed83hd.VR848348384dd-atjdjdfjdfjsnsn-hhsdjsdjsjdjsdj-wuUb0-kb88usdusdjsjdjsdjsdsd-dddddss-kjdhdhfhdhfhhdfh3he34h",
  "refresh_token": "5385267f-24a8-4fcf-9561-0380602868e2.Z8vxeBZ6TXXOmgsc8GUALnN9puxuqcqy",
  "token_type": "Bearer"
}
```

* The `refresh_token` field contains the refresh token.
* `expires_in` is how long (seconds) an access token is valid before it expires.

The following steps show an example of how to get a refresh token from Authgear with some common languages and frameworks.

### Step 1: Add Authgear Configuration to Your Code

For this step, log in to the Authgear Portal, navigate to the **Applications** section, and select your application. copy the configuration details for your application into your code as shown below:

{% tabs %}
{% tab title="JavaScript (React)" %}
For this React example, we are using the official Authgear JavaScript SDK. Install it using the following command:

```sh
npm install @authgear/web
```

```typescript
import authgear from "@authgear/web";

export const endpoint = ""; // The Authgear endpoint of your project e.g. https://my-app.authgearapps.com
export const clientID = ""; // Client ID can be obtained in the "Applications" page of the Portal

async function init() {
  try {
    await authgear.configure({
      endpoint,
      clientID,
      sessionType: "refresh_token"
    });
  } finally {
    createRoot(document.getElementById("react-app-root")!).render(<App />);
  }
}

// eslint-disable-next-line no-console
init().catch((e) => console.log(e));
```

Because the SDK can handle the task of getting a refresh token and using it to renew an access token, you do not need to specify the offline `scope` in your configuration.
{% endtab %}

{% tab title="PHP" %}
For this PHP example, we use the [League OAuth 2.0 client](https://oauth2-client.thephpleague.com/) PHP package. Install the package using the following command:

```sh
composer require league/oauth2-client
```

Then configure your application like this:

```php
<?php
require 'vendor/autoload.php';
session_start(); 

$appUrl = "https://your_project.authgear.cloud";
$clientID = "";
$clientSecret = "";
$redirectUri = "http://localhost:8081/";

$provider = new \League\OAuth2\Client\Provider\GenericProvider([
    'clientId'                => $clientID,    // The client ID assigned to you by the provider
    'clientSecret'            => $clientSecret,    // The client password assigned to you by the provider
    'redirectUri'             => $redirectUri,
    'urlAuthorize'            => $appUrl . '/oauth2/authorize',
    'urlAccessToken'          => $appUrl . '/oauth2/token',
    'urlResourceOwnerDetails' => $appUrl . '/oauth2/userInfo',
    'scopes' => 'openid offline_access'
]);
```

Notice the line with `'scopes' => 'openid offline_access'`, the `offline_access` scope is required for Authgear to return a refresh token.
{% endtab %}

{% tab title="Express.js" %}
For this example, we're using Node.js, and creating a simple Express application that makes HTTP requests to the Authgear server. Run the following commands to install the required Node package to run the code:

1\. Express:

```sh
npm install express
```

2\. Axios:

```sh
npm install axios
```

Now configure Authgear in your Express application like this:

```javascript
const express = require('express');
const axios = require('axios');

const app = express();
const port = process.env.PORT || 3000;

const config = {
  client: {
    id: "CLIENT_ID",
    secret: "CLIENT_SECRET",
    redirect_url: "REDIRECT_URL"
  },
  auth: {
    tokenHost: "AUTHGEAR_ENDPOINT",
    tokenPath: '/oauth2/token',
    authorizePath: '/oauth2/authorize',
    scope: 'openid+offline_access'
  },
};
```

The value in `config.auth.scope` includes the `offline_access` scope required for Authgear to return a refresh token when your application makes a call to the token endpoint.
{% endtab %}
{% endtabs %}

### Step 2: Get Refresh Token From Token Endpoint

Authgear will return the refresh token along with the access token when your application makes an HTTP request to the `/oauth2/token` endpoint, provided you include the `offline_access` scope as shown in step 1.

> Note: Authgear will omit refresh token if your initial request to the authorize URL does not include the **offline\_access** scope.

The following code examples show how to make a request to the token endpoint and get the refresh token from the response JSON object.

{% tabs %}
{% tab title="JavaScript (React)" %}
As mentioned earlier the Authgear JavaScript SDK can handle the task of getting the refresh token and using it to renew the access token when necessary. The following code shows how to call the method:

```javascript
authgear
    .refreshAccessTokenIfNeeded()
    .then(() => {
        // access token is ready to use
        // accessToken can be string or undefined
        // it will be empty if user is not logged in or session is invalid
        const accessToken = authgear.accessToken;

        // include Authorization header in your application request
        const headers = {
            Authorization: `Bearer ${accessToken}`
        };
    });
```
{% endtab %}

{% tab title="PHP" %}
While using the League OAuth client, you can get the refresh token returned by the token endpoint by calling the `getRefreshToken()` method on the object returned by the `provider->getAccessToken()`. The following code shows the flow for starting an authorization request, exchanging the authorization code for an access token, and getting the refresh token:

```php
if (!isset($_GET['code'])) {
    // Fetch the authorization URL from the provider; this returns the
    // urlAuthorize option and generates and applies any necessary parameters
    // (e.g. state).
    $authorizationUrl = $provider->getAuthorizationUrl();

    // Get the state generated for you and store it to the session.
    $_SESSION['oauth2state'] = $provider->getState();

    // Redirect the user to the authorization URL.
    header('Location: ' . $authorizationUrl);
    exit;
}
else {
    $code = $_GET['code'];

    if (empty($_GET['state']) || empty($_SESSION['oauth2state']) || $_GET['state'] !== $_SESSION['oauth2state']) {
        if (isset($_SESSION['oauth2state'])) {
            unset($_SESSION['oauth2state']);
        }

        exit('Invalid state');
    } else {
        try {
            $accessToken = $provider->getAccessToken('authorization_code', [
                'code' => $code
            ]);
            
            //Get refresh token
            $refreshToken = $accessToken->getRefreshToken();
            echo "Login successful access token:". $accessToken;
            echo " Refresh token:". $refreshToken;
            
        } catch (\League\OAuth2\Client\Provider\Exception\IdentityProviderException $e) {
            // Failed to get the access token or user details.
            exit($e->getMessage());
        }
    }
}
```
{% endtab %}

{% tab title="Express.js" %}
To get the refresh token in this Express example, simply send an HTTP request to the token endpoint to exchange the authorization for an access token. The refresh token will be in the response for the endpoint under a `refresh_token` field. The following code shows the example for reading the value of the `refresh_token` field in an express app:

```javascript
const data = {
  client_id: config.client.id,
  client_secret: config.client.secret,
  code: req.query.code,
  grant_type: 'authorization_code',
  response_type: 'code',
  redirect_uri: config.client.redirect_url,
  scope: config.auth.scope
};

try {
  const getToken = await axios.post(`
    ${config.auth.tokenHost}${config.auth.tokenPath}`,
    data,
    {
    headers: { "Content-Type": "application/x-www-form-urlencoded" }
    }
  );

  const accessToken = getToken.data.access_token;
  const refreshToken = getToken.data.refresh_token;
  res.send(`
  <p>${JSON.stringify(getToken.data)}</p>
    <div style="max-width: 650px; margin: 16px auto; background-color: #EDEDED; padding: 16px;">
      <p>Welcome</p>
      <p>Refresh token: ${refreshToken}</p>
      
    </div>
`);
} catch (error) {
  res.send("An error occurred! Login could not complete. Error data: " + error);
}
```
{% endtab %}
{% endtabs %}

## Summary

To get a refresh token from Authgear, you need to include `offline_access` in your authorization request.

The Authgear SDKs take care of getting `refresh token` and you do not need to do any extra configuration.

It is recommended to store the refresh token securely on the user's device and use it to request a new access token when the current one expires.
