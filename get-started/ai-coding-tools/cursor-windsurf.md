# Cursor/Windsurf

[![LLM | View as markdown](https://img.shields.io/badge/LLM-View%20as%20markdown-blue)](https://r.jina.ai/https://docs.authgear.com/get-started/ai-coding-tools/cursor-windsurf)

AI-assisted IDEs like Cursor and Windsurf are increasingly popular among developers for enhancing productivity. These tools offer LLM-based agents for code suggestions, debugging, and understanding codebases.

Now, you’ve created an innovative software project consisting of a frontend application and a backend server. By leveraging generative AI with the right prompts, you can integrate robust security features into your project within seconds.

## Create a project and an application on Authgear

First, create an account and a project in the [the Authgear portal](https://portal.authgear.com/).

In the project creation wizard, choose how your users will perform signup and login. For example:

* By email with a password
* By receiving OTP via SMS or WhatsApp

Next, navigate to the “Applications” page and create an application of the type **Single Page Application**.

There are two values you will need for the subsequent steps:

* **Client ID**: An ID to identify your application application with Authgear
* **Endpoint:** The URL to identify your Authgear project and allow your application to connect to it.

Under “Authorized Redirect URIs,” add the URL of your local environment with `/auth-redirect` appended. For example, if your frontend application runs on port 4000, use: `http://localhost:4000/auth-redirect` there.

Save your changes.

## Add user login to frontend code

To integrate login functionality into your frontend code, follow the corresponding documentation based on your framework:

* React: [https://docs.authgear.com/get-started/single-page-app/react](https://docs.authgear.com/get-started/single-page-app/react)
* Vue: [https://docs.authgear.com/get-started/single-page-app/vue](https://docs.authgear.com/get-started/single-page-app/vue)
* Angular: [https://docs.authgear.com/get-started/single-page-app/angular](https://docs.authgear.com/get-started/single-page-app/angular)

In the chat, select the documentation as context and put in the following prompt.

```
Use `@authgear/web` package, Integrate Authgear as Authentication provider

Store these in .env for initializing Authgear:

- Authgear Client ID is [CLIENT ID]
- Authgear Endpoint is [Authgear Endpoint]
- Redirect URL is http://localhost:4000/auth-redirect
```

To show the login status in the home page, use the following prompt to change the appearance of the logged in status and add a button to the [pre-built user settings page](../../customization/ui-customization/built-in-ui/auth-ui.md).

{% code overflow="wrap" %}
```
When logged in, show user id (sub) in user info, a logout button and a User settings button in the Home page
User Settings page is opened by `authgear.open(Page.Settings);`
```
{% endcode %}

Run the SPA with this prompt to verify the result.

```
Run dev frontend server at port 4000
```

## Add protection to backend code

In the backend, add the following documentation as context:

* Validate JWT in your application server: [https://docs.authgear.com/get-started/backend-api/jwt](https://docs.authgear.com/get-started/backend-api/jwt)

<details>

<summary>Additional context if the above didn't work</summary>

If the IDE failed to fetch the information from the documentation link, paste in the following as context.

**Note**: The prompt is designed for Express JS (Node JS) backend. For other backend technologies, copy and paste the corresponding code blocks from [jwt.md](../backend-api/jwt.md "mention") for the best result.

<pre data-overflow="wrap"><code><strong># Use JWKS to verify the JWT
</strong>
**Find the JWKS Endpoint**

Use the following method to get the JWKS URI (you'll need to URI to extract the public signing key from a JWT).

```
const appUrl = ""; //place your authgear app endpoint here
const getJwksUri = async (appUrl) => {
    const config_endpoint = appUrl + "/.well-known/openid-configuration";
    const data = await axios.get(config_endpoint);
    return data.data.jwks_uri;
}
```

**Extract JWT from Request Header**

Use the following code to extract only the token part from a `Bearer [token]` authorization header in your Express app:

```
const express = require("express");
const axios = require("axios");
const node_jwt = require('jsonwebtoken');
const jwksClient = require('jwks-rsa');

const app = express();
const port = 3002;
app.get('/', async (req, res) => {

    const requestHeader = req.headers;
    if (requestHeader.authorization == undefined) {
        res.send("Invalid header");
        return;
    }
    const authorizationHeader = requestHeader.authorization.split(" ");
    const access_token = authorizationHeader[1];

}
```

**Decode Access Token**

Next, decode the access token so that you can extract the JWT `kid` from the result. You'll need this `kid to get the public signing key. Use the following code to decode the JWT:

```
const decoded_access_token = node_jwt.decode(access_token, {complete: true});
```

**Get JWT Signing Keys and Verify the JWT**

Use the following code to extract the JWT public keys then verify the JWT using the keys:

```
const jwks_uri = await getJwksUri(appUrl);
    const client = jwksClient({
        strictSsl: true,
        jwksUri: jwks_uri
    });
    const signing_key = await client.getSigningKey(decoded_access_token.header.kid);

    try {
        const verify = node_jwt.verify(access_token, signing_key.publicKey, { algorithms: ['RS256'] });
        res.send(JSON.stringify(verify))
    }
    catch(error) {
        res.send(error);
    }
```
</code></pre>

</details>

{% code overflow="wrap" %}
```
In the backend, create a protected api `/me`,
When user access with a JWT access token issued by Authgear, return the user id, if not, return 401 unauthorized error
```
{% endcode %}

Run the backend server with this prompt to verify the result.

```
Run dev backend server at port 3000
```

Similarly you can prompt the chat to protect any API endpoints in the backend.

## Protect the API calls in the frontend

Now you have the a protected API in the backend, add a button in the frontend to test it out.

Include the corresponding frontend SPA docs to the "context" added in the previous step. Then use the following prompt to add a button the test the API call:

{% code overflow="wrap" %}
```
In the frontend, show a button to test the protected api `/me` both before and after logout. Use the `authgear.fetch("URL")` to call the API.
```
{% endcode %}

Run both the frontend and backend servers simultaneously. Your frontend should now feature a button that calls the protected API. Logged-in users will see their user ID retrieved successfully, while logged-out users will encounter an error message.
