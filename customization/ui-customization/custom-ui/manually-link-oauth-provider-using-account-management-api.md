---
description: Use the Account Management API to link OAuth provider.
---

# Manually Link OAuth Provider using Account Management API

The Management API provides a way to perform actions that you usually will do from the default `/settings` page using your own custom UI.

## Account Management API for OAuth Provider

The account management API for OAuth providers allows developers to write their own custom code for linking a social or enterprise (OAuth) provider to user accounts on their Authgear provider. Developers can use this API to create a custom UI that their users can use to link one or more social login providers to their accounts.

The account management API for OAuth Provider has 2 endpoints that your code must make HTTP(s) POST requests to. A detailed description of the endpoints is given below:

### 1. /api/v1/account/identification

Use this endpoint to initiate the flow of linking an OAuth provider to an account. The endpoint requires a valid access token for the current user in the Bearer Authorization header.

**Request Method:**

`POST`

**Request Header:**

```json
{
    "Authorization": "Bearer " + <ACCESS_TOKEN>
}
```

**Request Body:**

```json
{
    "identification": "oauth",
    "alias": "google",
    "redirect_uri": "http://localhost:3000/linkcallback",
    "exclude_state_in_authorization_url": false
}
```

* `identification`: Required. It must be the value `oauth`.&#x20;
* `alias`: Required. The alias of the OAuth provider you want the current account to associate with.&#x20;
* `redirect_uri`: Required. You have to specify your own redirect URI to your app or your website to receive the OAuth callback.&#x20;
* `exclude_state_in_authorization_url`: Optional. The default is `false`. When it is `false`, the `authorization_url` has a `state` parameter included, the token is bound to this `state` parameter. When it is `true`, the `authorization_url` has no `state` parameter included, the token is NOT bound to `state`. If you wish to use your own state, you must set this field to `true`.

**Response:**

```json
{
  "result": {
    "token": "oauthtoken_abc123xyz",
    "authorization_url": "https://www.google.com?client_id=client_id&redirect_uri=redirect_uri"
  }
}
```

* `token`: You store this token. You need to supply it after the end-user returns to your app.
* `authorization_url`: You MUST redirect the end-user to this URL to continue the authorization code flow. If `exclude_state_in_authorization_url` is false, it has the `state` parameter included.

### 2. /api/v1/account/identification/oauth

Call this endpoint from your redirect URI to finish the process of linking an OAuth provider.

**Request Method:**

`POST`

**Request Header:**

```json
{
    "Authorization": "Bearer " + <ACCESS_TOKEN>
}
```

**Request Body:**

```json
{
    "token": <TOKEN>,
    "query": <URL_QUERY>
}
```

* `token`: The token you received in the response of the [/api/v1/account/identification](manually-link-oauth-provider-using-account-management-api.md#id-1.-api-v1-account-identification) endpoint.&#x20;
* `query`: The query of the redirect URI.

**Response:**

```json
{"result":{}}
```

## How to Link OAuth Provider

In this example, we'll link Google as the OAuth provider.

### Step 1: User Login (Get Access Token)

The account manage API requires a user's valid access token to work. Hence, you must first authenticate the user using another identity first.

Set up your application to allow users to log in using Authgear and obtain an access token.

See our [Getting Started](broken-reference) section for a detailed guide for your preferred programming language or framework.

### Step 2: Initiate OAuth Link Flow

Now that your application can obtain a valid access token for a user, you'll make an HTTP(s) request to the `/api/v1/account/identification` endpoint to initiate the OAuth provider linking flow.

Use the following code to send the HTTP(s) request:

```javascript
app.get("/link", async (req, res) => {
    const endpoint = `<YOUR_AUTHGEAR_ENDPOINT>/api/v1/account/identification`;
    const accessToken = req.session.accessToken; //TODO Get access token for user here. The example retrieves the value stored in session cookies
    
    try {
    
        const initiateAccountLink = await axios.post(endpoint, 
        {
            "identification": "oauth",
            "alias": "google",
            "redirect_uri": `http://localhost:3000/linkcallback`,
            "exclude_state_in_authorization_url": false
        }, {
            headers: { "Authorization": "Bearer " + accessToken }
        });
        req.session.token = initiateAccountLink.data.result.token;
        
        res.send("linking initiated!"); // TODO Replace this link with Login with provider link instead
    } catch(error) {
        let errorInfo = error;
        if (error.response) {
            errorInfo = error.response.data;
        }
        res.send("An error occurred " + JSON.stringify(errorInfo));
    }

});
```

In the code above, the body of the HTTP(S) request includes the `identification` type which is `oauth` because we're trying to add a new OAuth Provider as an identity for a user's account. The value of `alias` is `google` because the OAuth provider to be added is Google.

The link in `redirect_uri` is a page on the example application that will handle the callback from the OAuth provider. We will implement this page in the next step.

The response from the `/api/v1/account/identification` endpoint should look like this:

```json
{
  "result": {
    "token": "oauthtoken_abc123xyz",
    "authorization_url": "https://www.google.com?client_id=client_id&redirect_uri=redirect_uri"
  }
}
```

### Step 3: Get OAuth Provider's Authorization URL

The response for the HTTP(s) request in Step 2 includes an authorization\_url field. That field contains the OAuth provider's authorization URL. Your application can redirect users to `authorization_url` for them to grant authorization on the OAuth provider's website.

Replace the line with `// TODO` in Step 2 with the following code to use the value of `authorization_url` in a hyperlink that users can click:

```javascript
res.send(`
    <a href="${initiateAccountLink.data.result.authorization_url}">Link Google Account</a>
    `);
```

### Step 4: Finish Linking Account

The final step for linking an OAuth provider is to implement the callback URL that the OAuth provider will be redirecting users back to after authorization. This is where your application will be making an HTTP(S) request to the second endpoint (`/api/v1/account/identification/oauth`).

For this example, we'll create a "`/linkcallback`" route to handle the redirect using the following code:

```javascript
app.get('/linkcallback', async (req, res) => {
    const endpoint = `<YOUR_AUTHGEAR_ENDPOINT>/api/v1/account/identification/oauth`;
    const accessToken = req.session.accessToken;
    const currentUrl = `${req.protocol}://${req.get('host')}${req.originalUrl}`;
    const queryParams = new URL(currentUrl).search;;
    
    try {
        const finishLink = await axios.post(endpoint, {
            "token": req.session.token,
            "query": queryParams,
        }, 
        {
            headers: { "Authorization": "Bearer " + accessToken }
        });
    
        res.send(JSON.stringify(finishLink.data));
    } catch(error) {
        let errorInfo = error;
        if (error.response) {
            errorInfo = error.response.data;
        }
        res.send("An error occurred " + JSON.stringify(errorInfo));
    }

});
```

The above code sends the `token` (returned in the response to the previous HTTP(S) request) and the `query` parameters added to the redirect URI by the OAuth provider in an HTTP(S) request to the finish endpoint.  Both parameters are required.

The following will be the response of the endpoint when the OAuth provider is linked successfully:

```json
{"result":{}}
```
