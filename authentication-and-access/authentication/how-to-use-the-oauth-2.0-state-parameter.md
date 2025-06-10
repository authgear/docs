---
description: >-
  Reference on what the OAuth 2.0 parameter is and how to use it in Authgear
  SDK.
---

# Use the OAuth 2.0 State Parameter

The [OAuth 2.0 framework](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1.1) includes an optional `state` parameter. The value of the `state` parameter can be any random string or number defined by a client application (e.g. a web or mobile that uses Authgear for user authentication) before making an authorization request. In fact, the `state` parameter is added to the authorization URL as a URL query.&#x20;

The authorization server (Authgear) will include the value of the state parameter when redirecting the user-agent back to the client application. As a result, the client application can retrieve the value of `state` returned to verify that it is the origin of the authorization request.

In this post, we'll cover some possible usage of the `state` parameter and how to include the `state` parameter in an authorization request to the Authgear server.

## Use cases of the State Parameter

The following are some use cases of the OAuth 2.0 state parameter.

### 1. Customize Post Login/Sign up User Experience

Because the value for the state parameter passed at the beginning of an authorization request is returned unchanged after authorization, you can use this behavior to customize the post-login or sign-up user experience.&#x20;

For example, you can show users some custom messages after they sign up or log in, using a special link that was sent to them via email or SMS. The "special" thing in the link would be the value of a query parameter that can be passed in the `state` parameter.

Then, a client application can read the value of the `state` parameter and based on that, determine when and how to display the custom message or user experience.

### 2. Analytics

Another possible use of the `state` parameter is analytics and tracking user behavior. You can use the state token to include a unique key that tracks your campaigns. This way, you can know the number of users who sign up or log in to your application from a particular campaign.

You can also use the value you specify in the `state` parameter in an analytic tool [Mixpanel](https://docs.mixpanel.com/docs/tracking-methods/id-management/identifying-users) (for example, as `id` in the `identify(id)` function) to track user's behavior pre-login and post-login.

To learn more about using the state parameter for tracking user behavior, see our detailed guide [here](../../integration/track-user-before-and-after-signup.md).

### 3. Security: To Prevent Cross-site Request Forgery (CSRF)

Cross-site Request Forgery or short CSRF is a type of web security vulnerability where the attacker uses malicious means to trick a user into performing undesired actions on sites they use and trust. This type of attack usually targets users who are signed in and attempts to compromise access to their protected resources.

In OAuth, an attacker can perform a Cross-site Request Forgery using the client application's redirect URI. The attacker can trick a user into using a redirect URI that contains their authorization code or access token. Hence the user will end up using the access token and protected resources of the attacker. When they save new data using this access token, the attacker can also view them (as they are the original owner of the protected resources).

The official Authgear SDKs have mechanisms for protecting your applications from CSRF built into them.

However, if you are not using the official SDK, you can secure your application by generating a random hard-to-guess value on the client application and passing it in the `state` parameter. Your application should store this value securely on the user's client-side using session cookies or some other form of local storage. Then,  verify the `state` parameter in the redirect URI against the value stored locally to confirm that a user-agent is the origin of an authorization request before exchanging the authorization code for an access token.&#x20;

## Examples: Including the State Parameter in Authorization Request

The following URL shows an example of an authorization request URL:

```
https://your_project.authgear.cloud/login?client_id=your_authgear_app_client_id&redirect_uri=http%3A%2F%2Flocalhost%3A4000%2Fauth-redirect&state=12345678
```

As you can see from the above URL, `state` is a query parameter in addition to other parameters like the `client_id` and `redirect_uri`.

If you're constructing the authorization URL manually, you can include the state parameter by simply appending "`&state=random_state_value`" to the authorization URL.

Alternatively, if you're using any of the Authgear SDKs, you can use the built-in `state` field to set a value.&#x20;

The following code samples show the use of the state parameter with Authgear.

{% tabs %}
{% tab title="JavaScript (React)" %}
### Step 1: Set up a React Project to use Authgear

Create a new React project or use an existing project and configure the project to use Authgear. The following example is based on our [React example Git repository](https://github.com/authgear/authgear-example-react).

First, install the Authgear web SDK by running the following command:

```sh
npm install @authgear/web
```

Next, configure Authgear in your React Project's index.tsx file like this:

```typescript
import authgear from "@authgear/web";

export const endpoint = "https://your_project.authgear.cloud"; // The Authgear endpoint of your project e.g. https://my-app.authgearapps.com
export const clientID = ""; // Client ID can be obtained in the "Applications" page of the Portal

async function init() {
  try {
    await authgear.configure({
      endpoint,
      clientID,
      sessionType: "refresh_token",
    });
  } finally {
    createRoot(document.getElementById("react-app-root")!).render(<App />);
  }
}

// eslint-disable-next-line no-console
init().catch((e) => console.log(e));
```

### Step 2: Include State Parameter in Authorization Request

Set the `state` field in your call to the `startAuthentication()` method of the Authgear SDK to a random hard-to-guess value based on your use case.

```typescript
  const startLogin = useCallback(() => {
    authgear
      .startAuthentication({
        redirectURI: "http://localhost:4000/auth-redirect",
        prompt: PromptOption.Login,
        state: "12345678"
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
```

### Step 3: Read and Use the Value of State Returned After the Authorization

Implement the component that handles your OAuth 2.0 redirect like this:

```typescript
const AuthRedirect: React.FC = () => {
  const usedToken = useRef(false);

  const navigate = useNavigate();

  useEffect(() => {
    async function updateToken() {
      try {
        const u = new URL(window.location.href);
        const params = u.searchParams;
        const state = params.get("state") ?? undefined
        
        if (state !== undefined) {
          const initialState = "12345678"; // In a real app store the initial value on the client side using something like session cookies.
          //compare value of state returned in redirectURL to initial value set in startAuthentication()
          if (state === initialState) {
            //values match, do things log state to an analytic tool, set custom URL to navigate user to...
            console.log("state parameter match");
          } else {
            //the value for state param return does not match, do something like stopping the authentication
            console.log("state parameter dont match");
            return;
          }
        }
        
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
```

The above code will read the value of the state parameter returned in the redirect and compare it to the initial value.

For this example, when the initial value of the `state` parameter before authorization is not the same as the value returned in the redirectURL, we halt the authentication process.&#x20;
{% endtab %}

{% tab title="PHP" %}
### Step 1: Set up PHP Project

This example uses the [League OAuth 2.0 client](https://oauth2-client.thephpleague.com/) PHP package. Install the package using the following command:

```sh
composer require league/oauth2-client
```

Next, configure your PHP to use Authgear like this:

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
```

The League OAuth 2.0 client we are using in this example helps us generate random strings for the `state` parameter. In the above code, we store the value for the state generated in session on the line with `$_SESSION['oauth2state'] = $provider->getState()`;.

### Step 2: Read and Use the Value of State

Add an else block for the `if (!isset($_GET['code']))` condition with the following code:

```php
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
            echo "Login successful ". $accessToken;
            
        } catch (\League\OAuth2\Client\Provider\Exception\IdentityProviderException $e) {
            // Failed to get the access token or user details.
            exit($e->getMessage());
        }
    }
}
```

The above code will prevent your PHP application from exchanging an authorization code for an access token when the value of `state` stored in the PHP session is not identical to the state returned in the redirect URI. The usage demonstrated above can prevent CSRF attacks.
{% endtab %}
{% endtabs %}
