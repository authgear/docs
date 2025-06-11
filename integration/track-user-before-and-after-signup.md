---
description: Learn how to track a user that signs up from a particular source or campaign.
---

# How to Track User Before and After Signup?

In this guide, you'll learn how to track users from a particular source before and after they sign up. For example, knowing how many people sign up for your application through a signup link that you have included in a blog post or an email campaign.

### Prerequisites

To follow this guide, you need to have the following:

* A free Authgear account.
* An application in the Authgear Portal and note down the configuration of the application.
* A client application that is written in your preferred language or framework.

### What We'll Build

For this guide, we'll build a demo Express application that has the following features:

* Read a `source` URL parameter defined by you (the developer).
* Send the value of the `source` parameter to the Authgear authorization server using the OAuth 2.0 `state` parameter.
* Read the value of `state` returned after user authorization (sign up) and log the value in a database or analytics system like Mixpanel.

### What is the OAuth 2.0 State Parameter?

The [OAuth 2.0 framework](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1.1) includes an optional `state` parameter. The value of the `state` parameter can be any random string or number defined by a client application (e.g a web or mobile that uses Authgear for user authentication) before the client makes an authorization request. In simple terms, the `state` parameter is added to the authorization URL as a URL query.

The authorization server (Authgear) will include the same state parameter value when redirecting the user-agent back to the client application. As a result, the client application can retrieve the value of `state` returned to verify that it is the origin of the authorization request.

We will use the above behavior of the authorization process to track a user before and after they sign up.

## How to Track a User Who Signs Up from a Particular Source

The following steps show the steps for building a simple application that is capable of tracking users before and after they sign up.

### Step 1: Set Up Your App to Use Authgear

First, create a new project directory and open it. The demo application for this guide is a simple [Express](https://expressjs.com/) application that use Axios to make HTTP requests. Hence, install both packages using the following commands:

1\. Express:

```sh
npm install express
```

2\. Axios:

```sh
npm install axios
```

Next, inside the project directory, create a new **app.js** file then add the following code to the file:

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
    scope: 'openid offline_access'
  },
};
```

You can get the correct configuration values (CLIENT\_ID, CLIENT\_SECRET, REDIRECT\_URL, and AUTHGEAR\_ENDPOINT) from the **Applications** section of the Authgear Portal.

### Step 2: Add State Parameter to Authorization Request

In this step, we'll implement a `/login` route in the demo application. This route will support a `source` query parameter, for example, `/login?source=002`.

Add the following code to the end of **app.js** to implement the `/login` route:

```javascript
app.get("/login", (req, res) => {

  const url = new URL(`${config.auth.tokenHost}${config.auth.authorizePath}`);
  url.searchParams.set('client_id', config.client.id);
  url.searchParams.set('redirect_uri', config.client.redirect_url);
  url.searchParams.set('response_type', 'code');
  url.searchParams.set('scope', config.auth.scope);

  if (req.query.source != null) {
    url.searchParams.set('state', req.query.source);
  }

  res.redirect(url);
  
});
```

The above code will check if the source parameter is included in the HTTP request to the `/login` route. If there is a `source` parameter, the value of the source will be added to the authorization URL's state parameter.

The following is an example of a login URL that includes the source parameter:

```url
http://localhost:3000/login?source=002
```

### Step 3: Read the Value of State After Authorization

As described earlier, Authgear will return whatever value you put in the `state` parameter of the authorization URL. In the last step, we will read the value of the `source` parameter from the `/login` route and pass it to the `state` parameter. In this step, we'll read the `state` parameter that Authgear returns in the redirect URI for the initial value we passed (`source`).

To do this, we'll add a `/` route to resolve our redirect URI. Add the following code to the end of **app.js** to implement the route:

```javascript
app.get("/", async (req, res) => {

  if (req.query.code != null) {
    const data = {
      client_id: config.client.id,
      client_secret: config.client.secret,
      code: req.query.code,
      grant_type: 'authorization_code',
      response_type: 'code',
      redirect_uri: config.client.redirect_url,
      scope: config.auth.scope
    };

    const sourceFromState = req.query.state;

    try {
      const getToken = await axios.post(`
        ${config.auth.tokenHost}${config.auth.tokenPath}`,
        data,
        {
        headers: { "Content-Type": "application/x-www-form-urlencoded" }
        }
      );

      const accessToken = getToken.data.access_token;
      res.send(`
      <p>Access token: ${accessToken}, ${sourceFromState}</p>
    `);
    } catch (error) {
      res.send("An error occurred! Login could not complete. Error data: " + error);
    }
  }

  else {
    res.send(`
      <div style="max-width: 650px; margin: 16px auto; background-color: #EDEDED; padding: 16px;">
        <p>Hi there!</p>
        <p>This demo app shows you how to add user authentication to your Express app using Authgear</p>
          <p>Checkout <a href="https://docs.authgear.com">docs.authgear.com</a> to learn more about adding Authgear to your apps.</p>
        <a href="/login">Login</a>
      </div>
    `);
  }
});

app.listen(port, () => {
  console.log(`server started on port ${port}!`);
});
```

The `sourceFromState` constant holds the value for the state parameter in the redirect URI. You can save this value to a database to track that the user has successfully signed up using the link with the source value in your original campaign link. You may also send this link to an analytic tool like Mixpanel to track the user and source.

To run the demo app, run the following command in the terminal:

```sh
node app.js
```

Then, open `http://localhost:3000/login?source=002` on a browser. You can change 002 to any random string that you wish to use for tracking a source or campaign. Also, be sure to add `http://localhost:3000` as a redirect URL for your application in the Authgear Portal.
