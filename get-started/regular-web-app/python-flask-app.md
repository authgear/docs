---
description: Authentication for a Python web application
---

# Python Flask App

This guide demonstrates how to add authentication with Authgear to a Python web application built with the [Flask](https://palletsprojects.com/p/flask/) framework using the [Authlib](https://authlib.org/) OAuth library. The full source code for this sample project can be found on the [GitHub repo](https://github.com/authgear/authgear-example-python-flask).

### Learning objectives

You will learn the following:

* How to create an app on Authgear.
* How to enable Email-based login.
* Add sign-up and login features to the Flask app.

### **Prerequisites**

Before you begin, you'll need the following:

* A **free Authgear account**. [Sign up](https://accounts.portal.authgear.com/signup) if you don't have one already.
* Make sure that [Python](https://www.python.org/downloads/) 3.10 or above is installed on your machine.
* Download and Install [Pip](https://pip.pypa.io/en/stable/installation/) to manage project packages.

### Part 1: Configure Authgear

To use Authgear services, you’ll need to have an application set up in the Authgear [Dashboard](https://portal.authgear.com/). This setup allows users in Authgear to sign in to the Flask application automatically once they are authenticated by Authgear.

#### Step 1: Configure an application

To set up the application, navigate to the [Authgear Portal UI](https://portal.authgear.com/) and select **Applications** on the left-hand navigation bar. Use the interactive selector to create a new **Authgear OIDC Client application** or select an existing application that represents the project you want to integrate with.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Every application in Authgear is assigned an alphanumeric, unique client ID that your application code will use to call Authgear APIs through the Authlib client library in the Flask app. Record the generated Authgear Issuer `Domain` (for example, `example-auth.authgear-apps.com`), `CLIENT ID`, `CLIENT SECRET` from the output. You will use these values in Part 2 for the Flask app config.

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Step 2: Configure **Redirect URI**

An **Authorized Redirect URI** of your application is the URL that Authgear will redirect to after the user has authenticated in the Authgear to complete the authentication process. In our case, it will be a home page for our Flask and it will run at [http://localhost:3000](http://localhost:3000).

Set the following [http://localhost:3000/callback](python-flask-app.md#learning-objectives) to the **Authorized Redirect URIs** field. If not set, users will not be returned to your application after they log in.

#### Step 3: Choose a Login method

After you create the **Authgear app**, you choose how users need to **authenticate on the login page**. From the **Authentication** tab, navigate to **Login Methods**, you can choose a **login method** from various options including, by email, mobile, or social, just using a username or the custom method you specify. For this demo, we choose the **Email+Passwordless** approach where our users are asked to register an account and log in by using their emails. They will receive a One-time password (OTP) to their emails and verify the code to use the app.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### Part 2: **Create a Flask application**

Next, create a Flask application with a single page and routes for home, callback, login, and logout flows.&#x20;

#### Step 1: Configure an .env file <a href="#configure-your-env-file" id="configure-your-env-file"></a>

Start with creating a `requirements.txt` file in your project directory:

```
flask>=2.0.3
python-dotenv>=0.19.2
authlib>=1.0
requests>=2.27.1
```

Run `pip install -r requirements.txt` from your command-line interface to make these dependencies available to the Python project.

#### Step 2: Setup the application <a href="#configure-your-env-file" id="configure-your-env-file"></a>

Create a `server.py` file in the project directory that contains application logic. Add the necessary libraries the application uses.

```python
import json
from os import environ as env
from urllib.parse import quote_plus, urlencode

from authlib.integrations.flask_client import OAuth
from dotenv import find_dotenv, load_dotenv
from flask import Flask, redirect, render_template, session, url_for
```

Load the configuration `.env` file to use values such as `AUTHGEAR_CLIENT_ID AUTHGEAR_CLIENT_SECRET`, `AUTHGEAR_DOMAIN` and `APP_SECRET_KEY` in the app.

Configure Authlib to handle the application's authentication with Authgear based on OIDC:

```python
oauth = OAuth(app)

oauth.register(
    "authgear",
    client_id=env.get("AUTHGEAR_CLIENT_ID"),
    client_secret=env.get("AUTHGEAR_CLIENT_SECRET"),
    client_kwargs={
        "scope": "openid offline_access",
    },
    server_metadata_url=f'https://{env.get("AUTHGEAR_DOMAIN")}/.well-known/openid-configuration',
)
```

#### Step 3: Setup the application routes <a href="#setup-your-routes" id="setup-your-routes"></a>

When visitors to the app visit the `/login` route, they'll be redirected to Authgear to begin the authentication flow.

```python
@app.route("/login")
def login():
    return oauth.authgear.authorize_redirect(
        redirect_uri=url_for("callback", _external=True)
    )
```

Once users complete the login process using Authgear, they will be redirected back to the application's `/callback` route. This route ensures that the user's session is saved, so they won't need to log in again during subsequent visits.

```python
@app.route("/callback", methods=["GET", "POST"])
def callback():
    token = oauth.authgear.authorize_access_token()
    session["user"] = token
    return redirect("/")
```

**Refresh Token**

Calling the  `authorize_access_token()` method of the Flask Authlib package will include a refresh token in the token response, provided your Flask application has `offline_access` as one of the OAuth 2.0 scopes.

Authlib will also use the refresh token to obtain a new access token automatically when the current access token has expired.

**Logout**

The route `/logout` manages the user's logout process from the application. It clears the user's session within the app and momentarily redirects to Authgear's logout endpoint to guarantee a thorough session clearance. After this, users are navigated back to your home route (which we'll discuss shortly).

```python
@app.route("/logout")
def logout():
    session.clear()
    return redirect(
        "https://"
        + env.get("AUTHGEAR_DOMAIN")
        + "/oauth2/end_session"
    )
```

The home route will either display the details of a logged-in user or provide an option for visitors to sign in.

```python
@app.route("/")
def home():
    return render_template(
        "home.html",
        session=session.get("user"),
        pretty=json.dumps(session.get("user"), indent=4),
    )
```

#### Step 4:  Add UI page <a href="#setup-your-routes" id="setup-your-routes"></a>

Create a new sub-directory in the project folder named `templates`, and create a file `home.html`.

```html
<html>
  <head>
    <meta charset="utf-8" />
    <title>Authgear Flak Login Example</title>
  </head>
  <body>
    <div data-gb-custom-block data-tag="if">
    <h1>Welcome {{session.userinfo.name}}!</h1>
    <p><a href="/logout" id="qsLogoutBtn">Logout</a></p>
    <div><pre>{{pretty}}</pre></div>
    <div data-gb-custom-block data-tag="else"></div>
    <h1 id="profileDropDown">Welcome Guest</h1>
    <p><a href="/login" id="qsLoginBtn">Login</a></p>
    </div>
  </body>
</html>
```

#### Step 5: Run the application

Run the application from the project root directory:

`python server.py`

The application should now be accessible to open from a browser at [http://localhost:3000](http://localhost:3000/).

### Next steps

There is so much more you can do with Authgear. Explore other means of login methods such as using [Magic links](https://docs.authgear.com/strategies/email-login-link) in an email, [social logins](https://docs.authgear.com/strategies/how-to-setup-sso-integrations), or [WhatsApp OTP](https://docs.authgear.com/strategies/whatsapp-otp-login). For the current application, you can also [add more users](https://docs.authgear.com/strategies/user-identity-and-authenticator) from the Authgear portal.
