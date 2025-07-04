---
description: >-
  Follow this quickstart tutorial to add authentication to your React
  application
---

# React

[![LLM | View as markdown](https://img.shields.io/badge/LLM-View%20as%20markdown-blue)](https://raw.githubusercontent.com/authgear/docs/refs/heads/main/get-started/single-page-app/react.md)

Authgear helps you add user logins to your React apps. It provides a pre-built login page and user settings page that can accelerate your development process.

Follow this :clock1: **15-minute** tutorial to create a simple app using React with the Authgear SDK.

{% hint style="info" %}
**Check out and clone** [<mark style="color:orange;">**the Sample Project on GitHub**</mark>](https://github.com/authgear/authgear-example-react)**.**
{% endhint %}

## Setup Application in Authgear

To use Authgear's features, you'll need an account and a Project. Sign up for a free account at [https://portal.authgear.com/](https://portal.authgear.com/) and create a new Project to get started.

After that, we will need to create an Application in the Project Portal.

### Create an application in the Portal

To create a client application, go to **Applications** on the left menu bar in the Authgear Portal.

<figure><img src="../../.gitbook/assets/authgear-nav-applications.png" alt="" width="375"><figcaption><p>portal navigate to applications</p></figcaption></figure>

Next, click **⊕Add Application** in the top toolbar.

Enter the name of your application, e.g. "MyAwesomeApp", then select **Single Page Application** as the Application Type. Click the **Save** button to create the application.

### Configure Authorize Redirect URI

The **Authorized Redirect URI** is a URL in you application where the user will be redirected to after login with Authgear. In this path, make a **finish authentication** call to complete the login process.&#x20;

Go to the URI section of the Authgear client application you just created and add a new **Authorized Redirect URI**. For this tutorial, add `http://localhost:4000/auth-redirect` to Authorize Redirect URIs.

Click **Save** to keep all client app configuration changes before proceeding to the next steps.

<figure><img src="../../.gitbook/assets/image (39).png" alt="" width="375"><figcaption><p>Configure Authorized Redirect URIs</p></figcaption></figure>

## Add Authgear to React App

In this section, we'll create a simple React application and connect it to Authgear such that, users of the app will log in, view their user settings, and log out of their account.

### Step 1: Create a simple React project

Here are some recommended steps to scaffold a React project. You can skip this part if you are adding Authgear to an existing project. See [#step-2-install-authgear-sdk-to-the-project](react.md#step-2-install-authgear-sdk-to-the-project "mention") in the next section.

#### Create a new React project using Vite

Run the following command from your preferred folder to create a new React project with Vite:

```bash
npm create vite@latest my-app -- --template react-ts 
```

Next, run the following commands to open the new project directory and install the dependencies:

```sh
cd my-app
npm install
```

#### Change port

In the `package.json` file, update the value of `dev` field in the `script.dev` section to:

```
"dev": "vite --port 4000",
```

This will enable the `npm run dev` command run the app in development mode on port 4000.&#x20;

The file structure in your project should look like this now:

```bash
my-app
├── node_modules
│   └── (...)
├── package-lock.json
├── package.json
├── index.html
└── src
    ├── App.tsx
    └── main.tsx
```

Run `npm run dev` now to run the project and you will see the default "Vite + React" page when you open `http://localhost:4000` on a web browser.

### Step 2: Install Authgear SDK to the project

Run the following command within your React project directory to install the Authgear Web SDK

```bash
npm install --save --save-exact @authgear/web
```

In `src/main.tsx` , import `authgear` and call the `configure` function to initialize an Authgear instance on application loads.

```tsx
import { createRoot } from 'react-dom/client'
import App from './App.tsx'
import authgear from "@authgear/web"

async function init() {
  try {
    // configure Authgear container instance
    await authgear.configure({
      endpoint: import.meta.env.VITE_AUTHGEAR_ENDPOINT,
      clientID: import.meta.env.VITE_AUTHGEAR_CLIENT_ID,
      sessionType: "refresh_token",
    });
  } finally {
    createRoot(document.getElementById("root")!).render(<App />);
  }
}

init().catch((e) => {
  // Error handling
  console.error(e)
});
```

The Authgear container instance takes `endpoint` and `clientID` as parameters. They can be obtained from the configuration page for the application created in [#setup-application-in-authgear](react.md#setup-application-in-authgear "mention"). Create a `.env` file in the root directory of your project and add your Authgear client application configuration using the following fields:

```properties
VITE_AUTHGEAR_CLIENT_ID=<CLIENT_ID>
VITE_AUTHGEAR_ENDPOINT=<AUTHGEAR_ENDPOINT>
VITE_AUTHGEAR_REDIRECT_URL=http://localhost:4000/auth-redirect
```

It is recommended to render the app after `configure()` resolves. So by the time the app is rendered, Authgear is ready to use.&#x20;

{% hint style="info" %}
Run **`npm run dev`** now and you should see the default page again and no error message in the console if Authgear SDK is configured successfully
{% endhint %}

### Step 3: Implement the Context Provider

Since we want to reference the logged-in state everywhere in the app, let's put the state in a **context provider** with `UserProvider.tsx` in the `/src/context` folder.&#x20;

The `UserProvider.tsx` file will have an `isLoggedIn` boolean and a `setIsLoggedIn` function. The  `isLoggedIn` boolean state can be auto-updated using the `onSessionStateChange` callback. This callback can be stored in `delegate` which is in the local SDK container.&#x20;

```tsx
// src/context/UserProvider.tsx
import React, { createContext, useEffect, useState, useMemo } from "react";
import authgear from "@authgear/web";

interface UserContextValue {
  isLoggedIn: boolean;
}

export const UserContext = createContext<UserContextValue>({
  isLoggedIn: false,
});

interface UserContextProviderProps {
  children: React.ReactNode;
}

const UserContextProvider: React.FC<UserContextProviderProps> = ({
  children,
}) => {
  // By default the user is not logged in
  const [isLoggedIn, setIsLoggedIn] = useState<boolean>(false);

  useEffect(() => {
    // When the sessionState changed, logged in state will also be changed
    authgear.delegate = {
      onSessionStateChange: (container) => {
        // sessionState is now up to date
        // Value of sessionState can be "NO_SESSION" or "AUTHENTICATED"
        const sessionState = container.sessionState;
        if (sessionState === "AUTHENTICATED") {
          setIsLoggedIn(true);
        } else {
          setIsLoggedIn(false);
        }
      },
    };
  }, [setIsLoggedIn]);

  const contextValue = useMemo<UserContextValue>(() => {
    return {
      isLoggedIn,
    };
  }, [isLoggedIn]);

  return (
    <UserContext.Provider value={contextValue}>{children}</UserContext.Provider>
  );
};

export default UserContextProvider;
```

### Step 4: Implement the Auth Redirect

Next, we will add an "AuthRedirect" page for handling the authentication result after the user has been authenticated by Authgear.

First, install `react-router-dom` using the following command:

```sh
npm install --save-exact react-router-dom
```

Create the `AuthRedirect.tsx` component file in the `src/` folder.&#x20;

Call the Authgear `finishAuthentication()` function in the Auth Redirect component to send a token back to Authgear server in exchange for an access token and a refresh token. Don't worry about the technical jargons, `finishAuthentication()` will do all the hard work for you and save the authentication data.

When the authentication is finished, the `isLoggedIn` state from the UserContextProvider will be automatically set to `true`.  Finally, navigate back to root (`/`) which is our Home page.

The final `AuthRedirect.tsx` will look like this

```tsx
// src/AuthRedirect.tsx
import React, { useEffect, useRef } from "react";
import { useNavigate } from "react-router-dom";
import authgear from "@authgear/web";

const AuthRedirect: React.FC = () => {
  const usedToken = useRef(false);

  const navigate = useNavigate();

  useEffect(() => {
    async function updateToken() {
      try {
        await authgear.finishAuthentication();
      } finally {
        navigate("/");
        usedToken.current = true;
      }
    }

    if (!usedToken.current) {
      updateToken().catch((e) => console.error(e));
    }
  }, [navigate]);

  return <></>;
};

export default AuthRedirect;

```

{% hint style="info" %}
Since in React 18, useEffect will be fired twice in development mode, we need to implement a [cleanup function](https://beta.reactjs.org/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development) to stop it from firing twice. We will use an `useRef` Hook to stop the user token from being sent twice to the Authgear Endpoint.

Without a cleanup function, an`useEffect`Hook will be fired twice and hence `finishAuthentication()` will send the token back to Authgear Endpoint for two times, which the second one will result in "Invalid Token" error since the token can only be used once.
{% endhint %}

### Step 5: Add Routes and Context Provider to the App

Now, we will add a "Home" page. Create a `Home.tsx` component file the `src/` folder.&#x20;

Then import **Home** and **AuthRedirect** as routes. And Import **UserContextProvider** and wrap the routes with it.

Your final `App.tsx` should look like this:

<pre class="language-tsx"><code class="lang-tsx">// src/App.tsx
import React from "react";
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import Home from './Home';
import AuthRedirect from './AuthRedirect';
import UserContextProvider from './context/UserProvider';

const App: React.FC = () => {
  return (
    &#x3C;UserContextProvider>
      &#x3C;Router>
<strong>        &#x3C;Routes>
</strong>          &#x3C;Route path="/auth-redirect" element={&#x3C;AuthRedirect />} />
          &#x3C;Route path="/" element={&#x3C;Home />} />
        &#x3C;/Routes>
      &#x3C;/Router>
    &#x3C;/UserContextProvider>
  );
}

export default App;
</code></pre>

The file structure should now look like

```
src
├── App.tsx
├── AuthRedirect.tsx
├── Home.tsx
├── context
│   └── UserProvider.tsx
└── main.tsx
```

### Step 6: Add a Login button

First, we will import the Authgear dependency and the React Hook that we will use to `Home.tsx`. Then add the login button which will call `startAuthentication(ConfigureOptions)` through the `startLogin` on click callback. This will redirect the user to the login page.

```tsx
// src/Home.tsx
import React, { useEffect, useState, useCallback, useContext } from 'react';
import authgear, { PromptOption } from '@authgear/web';

const Home: React.FC = () => {
  const startLogin = useCallback(() => {
    authgear
      .startAuthentication({
        redirectURI: import.meta.env.VITE_AUTHGEAR_REDIRECT_URL,
        prompt: PromptOption.Login,
      })
      .then(
        () => {
          // started authentication, user should be redirected to Authgear
        },
        err => {
          // failed to start authentication
        }
      );
  }, []);
  return (
    <div>
      <h1>Home Page</h1>
      <div>
        <button onClick={startLogin}>Login</button>
      </div>
    </div>
  );
}

export default Home;

```

You can now run **`npm run dev`** and you will be redirected to the Authgear Login page when you click the Login button.

![User will be redirected to the Authgear login page by clicking the login button](../../.gitbook/assets/authgear-authui-login.png)

### Step 7: Show the user information

The Authgear SDK helps you get the information of the logged-in users easily.

In the last step, the user is successfully logged in, so let's try to print the user ID (sub) of the user on the Home page.

In `Home.tsx`, we will add a simple loading splash and a greeting message printing the Sub ID. We will add two conditional elements such that they are only shown when user is logged in. We can also change the login button to show only if the user is not logged in.&#x20;

Make use of `isLoggedIn` from the `UserContext` to control the components on the page. Fetch the user info by `fetchInfo()` and access its `sub` property.

```tsx
// src/Home.tsx  
import React, { useEffect, useState, useCallback, useContext } from "react";
import authgear, { PromptOption } from "@authgear/web";
import { UserContext } from "./context/UserProvider";

const Home: React.FC = () => {
  const [greetingMessage, setGreetingMessage] = useState<string>("");
  const [isLoading, setIsLoading] = useState<boolean>(false);
  const { isLoggedIn } = useContext(UserContext);

  useEffect(() => {
    async function updateGreetingMessage() {
      setIsLoading(true);
      try {
        if (isLoggedIn) {
          const userInfo = await authgear.fetchUserInfo();
          setGreetingMessage("The current User sub: " + userInfo.sub);
        }
      } finally {
        setIsLoading(false);
      }
    }

    updateGreetingMessage().catch((e) => {
      console.error(e);
    });
  }, [isLoggedIn]);

  const startLogin = useCallback(() => {
    authgear
      .startAuthentication({
        redirectURI: "http://localhost:4000/auth-redirect",
        prompt: PromptOption.Login,
      })
      .then(
        () => {
          // started authentication, user should be redirected to Authgear
        },
        (err) => {
          // failed to start authentication
        }
      );
  }, []);

  return (
    <div>
      <h1>Home Page</h1>
      {isLoading && "Loading"}
      {greetingMessage ? <span>{greetingMessage}</span> : null}
      {!isLoggedIn && (
        <div>
          <button type="button" onClick={startLogin}>
            Login
          </button>
        </div>
      )}
    </div>
  );
};

export default Home;
```

Run the app again, the User ID (sub) of the user should be printed on the Home page.

### Step 8: Add a Logout button

Now, let's add a Logout button that is displayed when the user is logged in.&#x20;

In `Home.tsx`, we will use conditional elements to show a Logout button only for a user that is currently logged in.

Find the following line in Home.tsx:

```tsx
{!isLoggedIn && (
        <div>
          <button type="button" onClick={startLogin}>
            Login
          </button>
        </div>
)}
```

&#x20;Add the following code on a new line just after the above line:

```tsx
{isLoggedIn && (
  <div>
    <button onClick={logout}>Logout</button>
  </div>
)}
```

Then, add the `logout` callback:

```tsx
const logout = useCallback(() => {
  authgear
    .logout({
      redirectURI: "http://localhost:4000/",
    })
    .then(
      () => {
        setGreetingMessage('');
      },
      (err) => {
        console.error(err);
      }
  );
}, []);
```

Run the app again, we can now log out by clicking the Logout button.

### Step 9: Open User Settings

Authgear provides a built-in UI for the users to set their attributes and change security settings.&#x20;

Use the `openURL` function to open the settings page at `<your_app_endpoint>/settings`

In `Home.tsx` Add a conditional link to the existing elements.

```tsx
{isLoggedIn && (
  <a target="_blank" rel="noreferrer" onClick={userSetting} href="#">
    User Setting
  </a>
)}
```

And add the `userSetting` callback:

```tsx
import authgear, { Page } from "@authgear/web";
const userSetting = useCallback((e: React.MouseEvent<HTMLAnchorElement>) => {
    e.preventDefault();
    e.stopPropagation();
    authgear.open(Page.Settings);
}, []);
```

This is the resulting `Home.tsx`:

```tsx
// src/Home.tsx
import React, { useEffect, useState, useCallback, useContext } from "react";
import { UserContext } from "./context/UserProvider";
import authgear, { Page, PromptOption } from "@authgear/web";

const Home: React.FC = () => {
  const [greetingMessage, setGreetingMessage] = useState<string>("");
  const [isLoading, setIsLoading] = useState<boolean>(false);
  const { isLoggedIn } = useContext(UserContext);

  useEffect(() => {
    async function updateGreetingMessage() {
      setIsLoading(true);
      try {
        if (isLoggedIn) {
          const userInfo = await authgear.fetchUserInfo();
          setGreetingMessage("The current User sub: " + userInfo.sub);
        }
      } finally {
        setIsLoading(false);
      }
    }

    updateGreetingMessage().catch((e) => {
      console.error(e);
    });
  }, [isLoggedIn]);

  const startLogin = useCallback(() => {
    authgear
      .startAuthentication({
        redirectURI: "http://localhost:4000/auth-redirect",
        prompt: PromptOption.Login,
      })
      .then(
        () => {
          // started authorization, user should be redirected to Authgear
        },
        (err) => {
          // failed to start authorization
          console.error(err);
        }
      );
  }, []);

  const logout = useCallback(() => {
    authgear
      .logout({
        redirectURI: "http://localhost:4000/",
      })
      .then(
        () => {
          setGreetingMessage("");
        },
        (err) => {
          console.error(err);
        }
      );
  }, []);

  const userSetting = useCallback((e: React.MouseEvent<HTMLAnchorElement>) => {
    e.preventDefault();
    e.stopPropagation();
    authgear.open(Page.Settings);
  }, []);

  return (
    <div>
      {/* eslint-disable-next-line react/forbid-elements */}
      <h1>Home Page</h1>
      {isLoading && "Loading"}
      {greetingMessage ? <span>{greetingMessage}</span> : null}
      {!isLoggedIn && (
        <div>
          <button type="button" onClick={startLogin}>
            Login
          </button>
        </div>
      )}
      {isLoggedIn && (
        <div>
          <button type="button" onClick={logout}>
            Logout
          </button>
          <br />
          <a target="_blank" rel="noreferrer" onClick={userSetting} href="#">
            User Setting
          </a>
        </div>
      )}
    </div>
  );
};

export default Home;
```

![Show the User ID, a link to User Settings and a logout button after login](<../../.gitbook/assets/spa-react-sample-screenshot (1).png>)

## Next steps, Calling an API

To access restricted resources on your backend application server, the HTTP requests should include the access token in their Authorization headers. The Web SDK provides a `fetch` function which automatically handles this, or you can get the token with `authgear.accessToken`.

#### Option 1: Using fetch function provided by Authgear SDK

Authgear SDK provides the `fetch` function for you to call your application server. This `fetch` function will include the Authorization header in your application request, and handle the process of refreshing an access token automatically. The `authgear.fetch` implements [fetch](https://fetch.spec.whatwg.org/).

```javascript
authgear
    .fetch("YOUR_SERVER_URL")
    .then(response => response.json())
    .then(data => console.log(data));
```

#### Option 2: Add the access token to the HTTP request header

You can get the access token through `authgear.accessToken`. Call `refreshAccessTokenIfNeeded` every time before using the access token, the function will check and make the network call to refresh the access token only if it is expired. Include the access token into the Authorization header of the application requests.

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
