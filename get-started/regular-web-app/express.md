---
description: Authentication for Express.JS apps with Authgear and OAuth2
---

# Express

Authgear makes it easy to add user authentication to a regular web app that is not powered by any framework. You can do this by implementing OAuth2 in your app with Authgear as the provider.

In this post, you'll learn how to add user authentication to an Express.js application using Authgear.

### What You Will Learn

* How to create an Authgear Application
* How to sign in with Authgear from an Express app using an authorization code.
* How to request user info from Authgear

### Pre-requisites

You'll need the following to follow along with this tutorial:

* Node.js Installed
* A free Authgear account. Sign up for one here.

### How to Add User Authentication to Express.js App using Authgear

For this tutorial, we'll be building a simple Express app that has the following features:

* A landing page with a login link that takes users to the login route.
* A `/login` route that redirects users to the Authgear OAuth authorization page.
* Logic that exchanges the authorization code from Authgear for an access token.
* A page that uses that access token to fetch user info from Authgear.

#### Step 1: Configure Authgear Application

In order to use Authgear as an OAuth identity provider in your application, you need to configure an Authgear project. This project gives you all the credentials that you'll be using to send requests from your application.

Now let's create a new application.

Log in to the Authgear portal, and select your project. From the project dashboard, navigate to the Applications section and enter your application details as shown below:

<figure><img src="../../.gitbook/assets/authgear-app-create-express-real (1).png" alt=""><figcaption></figcaption></figure>

Once you're done, click on Save to reveal the OAuth configuration. The application configuration page is where you can find your OAuth credentials like client ID, client secret, and a list of supported endpoints. Note down the configuration details as you'll use them later in your Express app.

<figure><img src="../../.gitbook/assets/authgear-app-config-express (1).png" alt=""><figcaption></figcaption></figure>

#### Step 2: Set Redirect URI

You need to provide one or more URLs within your app for Authgear to redirect to after user authorization.

To add a URL, scroll to the **URIs** and click on the Add button. Enter the full URL for the page you wish to redirect users to after login. For our example app for this post, we'll set this URL to `http://localhost:3000`.

#### Step 3: Create Express App

It is now time to set up the Express app that will be interacting with Authgear. To do that, create a new folder with the name "express-auth-example", this will be your Express project folder. Next, run the following command from the project folder :

`npm install express`

We'll be using the Axios library to make HTTP requests in the example app. Install Axios to your project by running this command:

`npm install axios`

Once Express is installed, create a new `app.js` file in the root of your project folder and add the following code to the file:

```javascript
const express = require('express');
const axios = require('axios');

const config = {
  client: {
    id: "",
    secret: "",
    redirect_url: "http://localhost:3000"
  },
  auth: {
    tokenHost: 'https://your-project.authgear.cloud',
    tokenPath: '/oauth2/token',
    authorizePath: '/oauth2/authorize',
  },
};

app.get("/", async (req, res) => {
  res.send(`
      <div style="max-width: 650px; margin: 16px auto; background-color: #EDEDED; padding: 16px;">
        <p>Hi there!</p>
        <p>This demo app shows you how to add user authentication to your Express app using Authgear</p>
          <p>Checkout <a href="https://docs.authgear.com">docs.authgear.com</a> to learn more about adding Authgear to your apps.</p>
        <a href="/login">Login</a>
      </div>
    `);
});

app.listen(3000, () => {
  console.log("server started!");
});
```

**Note:** Paste the correct values of client id, client secret, and redirect URL for your Authgear app inside the `client` object in the config variable. Also, set tokenHost to the hostname part of your Authgear app endpoint URL. That is the part before the first "/". For example, tokenHost for `https://example.authgear.cloud/oauth2/token` will be `https://demo-1-ea.authgear.cloud`.

Run your app.js file using the `node app.js` command. You should get a page like this on a web browser when you visit `localhost:3000`:

<figure><img src="../../.gitbook/assets/authgear-example-express-land.png" alt=""><figcaption></figcaption></figure>

#### Step 4: Add Login Endpoint

Here we'll be implementing a local login endpoint in our Express app. This endpoint will handle requests from the above login link.

Add a new route to your express app using the following code:

```javascript
app.get("/login", (req, res) => {
  res.redirect(`${config.auth.tokenHost}${config.auth.authorizePath}/?client_id=${config.client.id}&redirect_uri=${config.client.redirect_url}&response_type=code&scope=openid`);
});
```

Now if you save your code and restart your app, clicking on the login link should redirect to the Authgear authorization page.

<figure><img src="../../.gitbook/assets/authgear-auth-page-express (1).png" alt=""><figcaption></figcaption></figure>

#### Step 5: Exchange Authorization Code For Access Token

After the user logs in and grants authorization to your app on Authgear, they are redirected back to the redirect URL you specified earlier. In addition to this redirect, an authorization code is sent via a `code` URL query parameter.

In this step, we will be exchanging the authorization code for an access code that users can later use to access protected resources.

Update the code for the `app.get("/")` route to the following:

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
      scope: "openid",
    };

    try {
      const getToken = await axios.post(`${config.auth.tokenHost}${config.auth.tokenPath}`, data, {
        headers: { "Content-Type": "application/x-www-form-urlencoded" }
      });

      const accessToken = getToken.data.access_token;
      console.log(accessToken);

      res.send("Welcome");

    } catch (error) {
      console.log(error);
      res.send("An error occurred! Could not complete login. Error data: " + error);
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
```

The above code sends an HTTP POST request to the token endpoint. The authorization code we got from the previous step is sent along with other client credentials in the HTTP request body. The header should contain `"Content-Type": "application/x-www-form-urlencoded"`

A valid access token is returned in the response to the Axios response in `response.data.access_token`.

We can now use this access token to make authenticated requests to protected resources in our app or from Authgear endpoint. In the next step, we'll attempt to get the current user's info from Authgear using the access token.

#### Step 6: Get User Info

Authgear provides an endpoint where your application can request user info. This endpoint will return the user's details on Authgear like their email address, gender, full name, and more.

To get user info in our example app, in app.js, replace this line :

```javascript
res.send("Welcome");
```

with the following code:

```javascript
//Now use access token to get user info.
const getUserInfo = await axios.get(`${config.auth.tokenHost}/oauth2/userinfo`, { headers: { "Authorization": "Bearer " + accessToken } });
const userInfo = getUserInfo.data;
res.send(`
   <div style="max-width: 650px; margin: 16px auto; background-color: #EDEDED; padding: 16px;">
           <p>Welcome ${userInfo.email}</p>
           <p>This demo app shows you how to add user authentication to your Express app using Authgear</p>
           <p>Checkout <a href="https://docs.authgear.com">docs.authgear.com</a> to learn more about adding Authgear to your apps.</p>
       
   </div>
`);
```

This code sends another HTTP request, but this time to the user info endpoint and the request type is GET. The access token is sent as a _Bearer_ authorization header.

At this point, save all changes and restart the application. Try logging in all over again, at the end you should be greeted with "Welcome \[your email address]".

<figure><img src="../../.gitbook/assets/authgear-app-login-success (2).png" alt=""><figcaption></figcaption></figure>

### Conclusion

And there you have it, you've successfully added user authentication to your Express app using Authgear as the OAuth provider.

You can add so much more to your app with the new Authgear authentication, like protecting your own app endpoint with the access code. You can also store the access token securely to persist the user session using express-session and cookies.

Here's a link to the complete code for [our example code on Github](https://github.com/authgear/authgear-example-express).
