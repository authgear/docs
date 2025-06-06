---
description: >-
  Learn how to add authentication to any web page without using Authgear's SDKs
  with IIFE(Immediately-invoked Function Expression) bundle
---

# Add authentication to any web page

In this guide, you'll make a simple website server to host the SPA app using [ExpressJS](https://expressjs.com/). We'll also use it to serve our HTML page and any assets it needs, like JavaScript, CSS, and so on. You can also view a [full-source code](https://github.com/authgear/authgear-example-spa-js) on the GitHub repo.

#### Prerequisites

* Before we start, ensure you have Node.js installed in your system. If not, download and install it from the [official website](https://nodejs.org/en/download/).
* **An Authgear account:** You need an Authgear account to follow this guide. If you don't have one, you can[ create it for free](https://accounts.portal.authgear.com/signup) on the Authgear website.
* **A Registered App:** You need a registered application type (Single Page Application) in Authgear. Follow the [setup application](../../get-started/single-page-app/website.md#setup-application-in-authgear) guide and skip [installing the Authgear Web SDK](../../get-started/single-page-app/website.md#step-4-install-authgear-web-javascript-sdk) part. You will retrieve the Authgear Web SDK from Authgear's CDN using IIFE(Immediately-invoked Function Expression) bundle and reference a script in our HTML directly.

### Create a basic web server

Start with making a **new folder** on your computer to keep the app’s source code (In the example, we call it `authgear-spa-js-login`). Then, initialize a new NPM project by running the following command:

```bash
npm init -y
```

Next, we install two required packages:

```jsx
npm install express
```

Also, install [nodemon](https://npmjs.org/package/nodemon) so that our server can be restarted automatically on any code changes in dev mode:

```jsx
npm install -D nodemon
```

Next, open the `package.json` file and edit scripts entry to have `start` and `dev` commands like the below:

```json
{
  // ...
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  // ...
}
```

Now you can run the app in two modes: _prod and dev_.

For example, `npm run dev` will run the application using `nodemon`, monitoring for changes as we modify files.

### Creating server.js

Create a new file `server.js` in the root of the project and populate it with the following code:

```json
const express = require("express");
const { join } = require("path");
const app = express();

// Serve static assets from the /public folder
app.use(express.static(join(__dirname, "public")));

// Endpoint to serve the configuration file
app.get("/authgear_config.json", (req, res) => {
  res.sendFile(join(__dirname, "authgear_config.json"));
});

// Serve the index page for all other requests
app.get("/*", (_, res) => {
  res.sendFile(join(__dirname, "index.html"));
});

// Listen on port 3000
app.listen(3000, () => console.log("Application running on port 3000"));
```

### Create a basic HTML page

Create a `index.html` file in the root of the project and add the following content to the created file:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Authgear SPA SDK Sample</title>
    <link rel="stylesheet" type="text/css" href="/css/main.css" />
  </head>

  <body>
    <h2>SPA Authentication Sample</h2>
    <p>Welcome to our page!</p>
    <button id="btn-login" disabled="true" onclick="login()">Log in</button>
    <button id="btn-logout" disabled="true" onclick="logout()">Log out</button>
    <script src="js/app.js"></script>
    <script src="<https://unpkg.com/@authgear/web@2.2.0/dist/authgear-web.iife.js>"></script>
  </body>
</html>
```

We do not use a package manager such as [Webpack](https://webpack.js.org/), we will retrieve the Authgear Web SDK from Authgear's CDN using IIFE(Immediately-invoked Function Expression) bundle. We can reference a script in our HTML directly:

```html
<script src="<https://unpkg.com/@authgear/web@2.2.0/dist/authgear-web.iife.js>"></script>
```

> You can install the Authgear Web SDK as a dependency of your application, it is useful if you are building React or React Native apps. See how to [install the package](https://docs.authgear.com/get-started/single-page-app/website#install-the-authgear-web-sdk).

### Create a main.css file

Create a new folder called `public` folder in the project root folder and create another folder called `css` inside the `public` folder. Add a new file in there called `main.css`. This will be used to determine how the log-in and log-out button elements will be hidden on the main page depending on whether a user is authenticated or not.

Open the newly-created `public/css/main.css` file and add the following CSS:

```css
.hidden {
    display: none;
}
  
label {
    margin-bottom: 10px;
    display: block;
}
```

After creating an HTML file and applying CSS styles, see now how our page looks like by running `npm run dev` and accessing it at http://localhost:3000.



<figure><img src="../../.gitbook/assets/Untitled (8).png" alt=""><figcaption></figcaption></figure>

### Create an app.js file

To add some action to the page, we create a new directory in the `public` folder called `js`, and add a new file there called `app.js`. Copy and paste the following JS code that reads `authgear_config.json` file Authgear app-specific values (`endpoint` and `clientId`) from the endpoint using `fetchAuthConfig` function. Also, it configures a new Authgear client, and defines login and logout logic:

```jsx
let authgearClient = null;

const fetchAuthConfig = () => fetch("/authgear_config.json");

const configureClient = async () => {
    const response = await fetchAuthConfig();
    const config = await response.json();
    authgearClient = window.authgear.default;

    await authgearClient.configure({
        endpoint: config.endpoint,
        clientID: config.clientID,
        sessionType: "refresh_token",
    }).then(
        () => {
            console.log("Authgear client successfully configured!");
        },
        (err) => {
            console.log("Failed to configure Authgear");
        }
    );
};

const login = async () => {
    await authgearClient
        .startAuthentication({
            redirectURI: window.location.origin,
            prompt: "login",
        })
        .then(
            () => {
                console.log("Logged in!");
            },
            (err) => {
                console.log("Log in failed", err);
            }
        );
};

const logout = () => {
    authgearClient
    .logout({
      redirectURI: window.location.origin,
    })
    .then(
      () => {
        console.log("Logged out successfully");
      },
      (err) => {
        console.log("Failed to logout");
      }
    );
};

window.onload = async () => {
    await configureClient();
    updateUI();

    const query = window.location.search;
    if (query.includes("code=")) {

        updateUI();

        window.history.replaceState({}, document.title, "/");
    }
}

const updateUI = async () => {
    const isAuthenticated = authgearClient.sessionState === "AUTHENTICATED";

    document.getElementById("btn-logout").disabled = !isAuthenticated;
    document.getElementById("btn-login").disabled = isAuthenticated;
};
```

### Understanding the whole picture

Let’s breakdown down `app.js` code in the previous section and understand how authentication is achieved with Authgear:

**Configure the Authgear client**

`fetchAuthConfig`: Firstly, this function makes a request to the `/authgear_config.json` the endpoint we exposed in `server.js` to fetch Authgear app setting values from `authgear_config.json`file.

`configureClient`: Once we retrieve the configuration information for the Authgear client from the `authgear_config.json` file and we set up the Authgear client with these settings. It also logs a message to the console, informing whether the configuration was successful or not.

**Login flow**

`login`: The function is called by the **Login** button previously defined on the HTML page. It performs the login action by calling `authgearClient.startAuthentication` Authgear’s function. It redirects the user to the Auhthgear login page. After the user logs in successfully, they will be redirected back to the same page we set in `redirectURI`. Run the project and click the **Login** button. You should be taken to the **Authgear Login Page** configured for your application.&#x20;

<figure><img src="../../.gitbook/assets/Untitled (9).png" alt=""><figcaption></figcaption></figure>

Go ahead and create a new user or log in using an email (we specified the Passwordless Email login method in the first part). When you try to log in with your email, you should receive a [magic link](https://docs.authgear.com/strategies/email-login-link) to your email box to confirm login operation.

<div><figure><img src="../../.gitbook/assets/Untitled (11).png" alt=""><figcaption></figcaption></figure> <figure><img src="../../.gitbook/assets/Untitled (10).png" alt=""><figcaption></figcaption></figure></div>

After authenticating successfully, you will be redirected to the page you were before.

**Logout flow**

`logout`: This function logs the user out and redirects them back to the original page (at[http://localhost:3000](http://localhost:3000)). It uses Authgear’s `logout` function and logs a message to the console indicating the result of the operation.

**Update the UI**

`window.onload`: This is a function that runs when the page loads. It configures the Authgear client and updates the UI. If the page's URL contains a "code=" it means the user is authenticated (`code` the query will be received from the Authgear server), it updates the UI again and removes the "code=" from the URL.

**Evaluate the authentication state**

`updateUI`: This function updates the status of the login and logout buttons based on whether the user is authenticated or not. In Authgear, you can check if the user has logged in or not with `sessionState` the attribute. If the user is authenticated, we disable the login button and enable the logout button, and vice versa if the user is not authenticated.
