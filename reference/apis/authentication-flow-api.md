---
description: >-
  API reference detailing all endpoints, inputs and results supported by the
  Authentication Flow API.
---

# Authentication Flow API

Welcome to the API reference for the Authentication Flow API. On this page, you'll find a complete list of all the actions supported by the API.

For an overview of what the Authentication Flow API is, see this post [here](../../how-to-guide/custom-ui/authentication-flow-api.md).

## 1.0 Introduction

This section of the API reference documentation covers the following:

* Endpoints
* How to make your first HTTP request to the authentication flow endpoint.&#x20;
* Authentication Flow Types
* How state token works
* API inputs and responses

### 1.1 Endpoints

The Authentication Flow API has two endpoints. The endpoints are:

* `/api/v1/authentication_flows` :  This endpoint can be used to initialize a new authentication flow. Or to perform a flow that requires just one step/HTTP request to finish.
* `/api/v1/authentication_flows/states/input`: This endpoint requires a `state_token` parameter in the request body. The value for the `state_token` parameter must be a valid state token gotten from a previous step/request to the Authentication Flow API.

Both endpoints only support requests made over HTTPS  using the POST method. Requests made over plain HTTP will be refused. The supported content type for the request is `application/json`.&#x20;

### 1.2 Create a New Authentication Flow

The first step for using the Authentication Flow API is to create a new authentication flow.

#### What is an Authentication Flow?

An authentication flow basically starts with an HTTP request to the `/api/v1/authentication_flows` endpoint. The endpoint will respond with data such as a `state_token` and an `action` object that will determine what the next step (request) will be executed. The `action` object returns different values for its own `type` property (action.type) which include: `identify`, `authenticate`, `verify`, `finish_redirect_url`, etc.

At the minimum, this step involves sending an HTTP request to the Authentication Flow API endpoint with the `type` and `name` parameters in the request body.

**HTTP Request:**

Here's a sample of what the HTTP request to create a new flow of "`type"`: "`login"` will look like this:

```http
POST /api/v1/authentication_flows
Content-Type: application/json

{
  "type": "login",
  "name": "default"
}
```

The above code block shows how the structure of a typical HTTP request to the Authentication Flow API should be configured. Here's a breakdown of all the different parts of the request:

* The API expects a **POST** request sent over **HTTPS**.
* The endpoint to make an HTTP request to is **/api/v1/authentication\_flows**. An example of the full endpoint is `https://{your-project.authgear.cloud}/api/v1/authentication_flows`. Replace `{your-project.authgear.cloud}` with the valid domain name for your Authgear project.
* Always set the request content type as **Content-Type: application/json** as the API only accepts inputs as JSON.
* The JSON object containing values for **type** and **name** can then be passed in the request body.
* The `type` parameter defines the [authentication flow type](authentication-flow-api.md#id-1.3-authentication-flow-types) while the `name` parameter can be set to "default".

**HTTP Request for a Single-Step Authentication Flow:**

The following section shows an example request for a single-step authentication flow that uses the phone number + password login method.&#x20;

The Authentication Flow API supports the multiple login methods that you can enable for your project on the Authgear Portal. Hence, to use a single-step authentication flow like in the example below, ensure that the phone number + password login method is enabled and the user has it enabled for their account.

```http
POST /api/v1/authentication_flows
Content-Type: application/json

{
  "type": "login",
  "name": "default",
  "batch_input": [
        {
            "identification": "phone",
            "login_id": "+44123344556"
        },
        {
            "authentication": "primary_password",
            "password": "12345678"
        }
    ]
}
```

**Example:**

The following are examples of how to make an HTTP request to the API to start a login flow from your code:

{% tabs %}
{% tab title="Node.js" %}
```javascript
const express = require('express');
const axios = require('axios');

function rawURLQuery(url) {
    const index = url.indexOf('?');
    return url.substr(index + 1);
}

const app = express();
const port = process.env.PORT || 3000;

const endpoint = "https://your-project.authgear.cloud"; //replace with your actual Authgear project endpoint
async function userLogin(url_query) {
    const url = `${endpoint}/api/v1/authentication_flows?${url_query}`;

    const input = {
        "type": "login",
        "name": "default"
    };

    const headers = {
        "Content-Type": "application/json"
    }

    try {
        const startLogin = await axios.post(`${url}`, input, {
            headers: headers
        });

        return startLogin;

    }
    catch (error) {
        console.log(error.response);
        return error.response;

    }
}

app.get('/login', async (req, res) => {
    const url_query = rawURLQuery(req.url);
    const apiResponse = await userLogin(url_query);
    res.send(apiResponse.data);
});

app.listen(port, () => {
    console.log(`server started on port ${port}!`);
});
```
{% endtab %}

{% tab title="PHP" %}
```php
require 'vendor/autoload.php';
use GuzzleHttp\Client;

$endpoint = "https://your-project.authgear.cloud"; //replace with your actual Authgear project endpoint
$url_params = $_SERVER['QUERY_STRING'];
$url = $endpoint . "/api/v1/authentication_flows" . "?" . $url_params;

$client = new Client();
$headers = [
    'Content-Type' => 'application/json',
    'Accept' => 'application/json',
];
$response = $client->post($url, [
    "json" => [
        'type' => 'login',
        'name' => 'default'
    ],
    "headers" => $headers
]);

var_dump($response->getBody());
```
{% endtab %}
{% endtabs %}

**Note**: In order to get the `finish_redirect_url` at the end of your authentication flow, you must include the value of the entire URL Query generated by Authgear in your initial HTTP request for your authentication flow.  The URL Query is a set of parameters that the Authgear authorization server adds to your Custom UI URI when redirecting users to your custom authentication UI.

### 1.3 Authentication Flow Types

You can specify the kind of authentication flow you wish to initialize using the `type` parameter in your HTTP request body.

At the moment, the API supports these three values in the type parameter:

* `signup`: The flow to sign up as a new user.&#x20;
* `login`: The flow to sign in as a new user.
* `signup_login`: This flow will either become a signup or login depending on the input. If the end-user enters an existing login ID, then the flow will become login, otherwise, it is signup.
* `account_recovery`: You can use this type to create a flow like account recovery or password reset.

### 1.4 State Token

The state token can help you track the state of your authentication flow. Every successful response from the API endpoint includes a stake token. The following is an example of a state token in an HTTP response:

```json
"result": {
    "state_token": "authflowstate_ABCD1234EF56GH8A0KFC998K",
    "type": "login",
    "name": "default",
    "action": {
            "type": "identify",
            "data": {
                "options": [
                    {
                        "identification": "email"
                    }
                ]
            }
        }
}
```

You can include this state token in the body of your next HTTP request to continue the flow. Below is an example of a request that uses the state token in the above example to continue to the next step of the `login` type flow.

**HTTP Request:**

```http
POST /api/v1/authentication_flows/states/input
Content-Type: application/json

{
    "state_token": "authflowstate_ABCD1234EF56GH8A0KFC998K",
    "input": {
        "identification": "email",
        "login_id": "user@example.com"
    }
}
```

**Example:**

{% tabs %}
{% tab title="JavaScript" %}


```javascript
const express = require('express');
const axios = require('axios');

const app = express();
const port = process.env.PORT || 3000;

const endpoint = "https://your-project.authgear.cloud"; //replace with your actual Authgear project endpoint
async function submitEmailUserLogin(email, state_token) {
    
    const url = `${endpoint}/api/v1/authentication_flows/states/input`;

    const input = {
        "state_token": state_token,
        "input": {
            "identification": "email",
            "login_id": email
        }
    };

    const headers = {
        "Content-Type": "application/json"
    }

    try {
        const submitEmail = await axios.post(`${url}`, input, {
            headers: headers
        });

        return submitEmail;

    }
    catch (error) {
        console.log(error.response);
        return error.response;

    }
}

app.get('/loginStep2', async (req, res) => {
    const apiResponse = await submitEmailUserLogin("pius@oursky.com", "ENTER_VALID_AUTH_STATE");
    res.send(apiResponse.data);
});

app.listen(port, () => {
    console.log(`server started on port ${port}!`);
});
```
{% endtab %}

{% tab title="PHP" %}


```php
require 'vendor/autoload.php';
use GuzzleHttp\Client;

$endpoint = "https://your-project.authgear.cloud"; //replace with your actual Authgear project endpoint

$url = $endpoint . "/api/v1/authentication_flows/states/input";

$client = new Client();
$headers = [
    'Content-Type' => 'application/json',
    'Accept' => 'application/json',
];
$response = $client->post($url, [
    "json" => [
        "state_token" => "ENTER_VALID_AUTH_STATE",
        "input" => [
            "identification" => "email",
            "login_id" => "user@example.com"
        ]
    ],
    "headers" => $headers
]);

var_dump($response->getBody());
```
{% endtab %}
{% endtabs %}

### 1.5 Input

The Authentication Flow API supports inputs passed in the HTTP requests body as JSON. Some special fields such as `type`, `name` and `state_token` can be passed directly as parameters in the request body. However, other values like user inputs must be passed using the `input` or `batch_input` field.

In order to pass inputs that modify the state of the authentication flow, you must make your HTTP requests to the `/api/v1/authentication_flows/states/input` endpoint instead of `/api/v1/authentication_flows`.

We've provided the structure for a request with inputs below:

**HTTP Request:**

```http
POST /api/v1/authentication_flows/states/input
Content-Type: application/json

{
    "state_token": "authflowstate_ABCD1234EF56GH8A0KFC998K",
    "batch_input":
            [
                {
                    "identification": "email",
                    "login_id": "user@example.com"
                },
                {
                    "authentication": "primary_password",
                    "password": "12Hjdusd@o*qfhs$"
                }
            ]
}
```

From the above schema, notice how the state\_token parameter is a direct parameter in the request body. User inputs such as email address and password on the other hand are within the batch\_input parameter as elements in an array.

The keys for sending email and password are special words specified by Authgear. Usually, you'll find these keys in the response from the previous step of your authentication flow.

### 1.6 Response

The structure of the HTTP response from the Authentication Flow API tries to be as consistent as possible all the time. In other words, the structure of responses is always the same. This response is in JSON format.

Successful responses return a `200` HTTP status code and the response body look like this:

```json
{
    "result": {
        "state_token": "authflowstate_ABCD1234EF56GH8A0KFC998K",
        "type": "login",
        "name": "default",
        "action": {
            "type": "authenticate",
            "data": {
                "options": [
                    {
                        "authentication": "primary_password"
                    }
                ],
                "device_token_enabled": false
            }
        }
    }
}
```

* `state_token`: The token that refers to this particular state of an authentication flow.
* `type`: The type of the authentication flow. Possible values are`signup`, `login` and `signup_login`.
* `name`: The name of the authentication flow.
* `action`: An object containing information about the next step based on the current request. For example, if we send only `identification` information (for instance email) in the current request, `action.type` will be `authenticate` and `action.data` will contain information about the enabled authentication methods.
* `action.data.device_token_enabled`:  When the value of this field is `true`, you can send  `request_device_token: true` in your next input to skip 2FA. The action just described is equivalent to checking "Skip 2FA next time on this device" box in AuthUI. Authgear stores device token in the browser cookies to allow skipping of 2FA in the future.

#### 1.6.1 Error Response

The Authentication Flow API returns an HTTP status code within the 400 range when a request is not successful. The following is an example of the body of an error response:

```json
{
  "error": {
    "name": "Unauthorized",
    "reason": "InvalidCredentials",
    "message": "invalid credentials",
    "code": 401,
    "info": {}
  }
}
```

* `reason`: You can use this string to distinguish between different errors. Do NOT use `message` as it could change over time.
* `info`: An object containing extra information about the error. It can be absent (i.e. not `null`, but absent).

## 2.0 Actions (action.type)

Here, we'll provide details about all the supported action types (`action.type`) for signup flow and login flow.

### 2.1 action.type: identify (signup)

This is usually contained in the response from the first step of an authentication flow of `type: signup`.

**HTTP Request:**

```http
POST /api/v1/authentication_flows
Content-Type: application/json

{
  "type": "signup",
  "name": "default"
}
```

When you make the above HTTP request to create a signup flow, you get a response that looks like this:

```json
{
  "result": {
    "state_token": "authflowstate_5R6NM7HGGKV64538R0QEGY9RQBDM4PZD",
    "type": "signup",
    "name": "default",
    "action": {
      "type": "identify",
      "data": {
        "options": [
          {
            "identification": "email"
          },
          {
            "identification": "phone"
          },
          {
            "identification": "oauth",
            "provider_type": "google",
            "alias": "google"
          },
          {
            "identification": "oauth",
            "provider_type": "wechat",
            "alias": "wechat_mobile",
            "wechat_app_type": "mobile"
          }
        ]
      }
    }
  }
}
```

The `action.data.options` object in the response contains an array with [all the supported identification methods](authentication-flow-api.md#id-3.0-input-identifications). For example, the presence of `{ "identification": "email" }` means you can continue the above signup flow using an email address to identify the user.

You can select one of the options and then include it in the next step (request) of your authentication flow like this:

```json
"input": {
        "identification": "email",
        "login_id": "user@example.com"
    }
```

### 2.2 action.type: create\_authenticator

This action requires the UI to create a new authenticator. For example, you can use the `create_authenticator` action to set a new password for a new user during account creation (signup).

**Response:**

```json
{
    "result": {
        "state_token": "authflowstate_CCB9PT4EMVDKWKG7KW0ZVBG6JZ28DCK3",
        "type": "signup",
        "name": "default",
        "action": {
            "type": "create_authenticator",
            "data": {
                "options": [
                    {
                        "authentication": "primary_password",
                        "password_policy": {
                            "minimum_length": 8,
                            "history": {
                                "enabled": false
                            }
                        }
                    }
                ]
            }
        }
    }
}
```

From the above response `action.data.options` shows that we can use the `primary_password` authentication method. Learn more about all the authenticators supported by Authgear, in the [Authentications section](authentication-flow-api.md#id-4.0-input-authentications).

To set the primary password for the new user, send the following input in your next request:

```json
"input": {
  "authentication": "primary_password",
  "new_password": "12Hjdusd@o*qfhs$"
}
```

Sending the above input will complete the signup flow for signup using the primary\_password authentication. Hence, the value `action.type` in response to the next request will be "finished".

### 2.3 action.type: identify (login)

If you create an authentication flow of `type: login` without defining any of `input` or `batch_input`, the response to your HTTP request will include `action.type:identify`.&#x20;

Below is the structure of a simple HTTP request for creating a login flow.&#x20;

**HTTP Request**:

```http
POST /api/v1/authentication_flows
Content-Type: application/json

{
  "type": "login",
  "name": "default"
}
```

The response for the above HTTP request will look like this:

```json
{
    "result": {
        "state_token": "authflowstate_30AQP46AD1DR219C0SBRNP5NG066R8YG",
        "type": "login",
        "name": "default",
        "action": {
            "type": "identify",
            "data": {
                "options": [
                    {
                        "identification": "email"
                    },
                    {
                        "identification": "oauth",
                        "provider_type": "wechat",
                        "alias": "wechat_mobile",
                        "wechat_app_type": "mobile"
                    }
                ]
            }
        }
    }
}
```

`action.data.options` contains a list of all [supported identification methods](authentication-flow-api.md#id-3.0-input-identifications).

You can use one of the options ( "identification": "email") to continue the login flow by passing it in your next request using the input parameter like this:

```json
"input": {
        "identification": "email",
        "login_id": "user@example.com"
        }
```

### 2.4 action.type: authenticate

`actiontype.type:authenticate` is returned in the response for the step that passes an identification option in its request body. The response that includes this action looks like the following:

```json
{
    "result": {
        "state_token": "authflowstate_BAQ271GXCR3NNMR8DW3HTCWNET7TCQR8",
        "type": "login",
        "name": "default",
        "action": {
            "type": "authenticate",
            "data": {
                "options": [
                    {
                        "authentication": "primary_password"
                    }
                ],
                "device_token_enable": false
            }
        }
    }
}
```

The `action.data.options` object for the above response contains all the supported authenticators. As you can see, for the above example, only the `primary_password` authenticator is supported. To learn more about all the authenticators supported by Authgear, check out the [Authentications section](authentication-flow-api.md#id-4.0-input-authentications).

You can pass the authenticator value (user's primary password) in your next request using the input parameter like this:

```json
{
    "state_token": "authflowstate_J4SW4PGVXSZXS95NJAP8F67VZ31V4D0D",
    "input": {
        "authentication": "primary_password",
        "password": "12345678"
    }
}
```

For a simple login with a primary password only, sending the above request will return `action.type: finished` which means the authentication flow is completed.

### 2.5 action.type: verify

The response that includes this action may look like this:

```json
{
  "result": {
    "state_token": "authflowstate_PZMX4FG4N82WGSSY0Y398YH0F9BX4FPX",
    "type": "signup",
    "name": "default",
    "action": {
      "type": "verify",
      "data": {
        "channels": [
          "sms",
          "whatsapp"
        ]
      }
    }
  }
}
```

To send the OTP for the verification process, use the state token returned in the response to send the following input in your next request:

```json
"state_token": "authflowstate_PZMX4FG4N82WGSSY0Y398YH0F9BX4FPX",
"input": {
  "channel": "sms"
}
```

The response to the above request should look like this:

```json
{
  "result": {
    "state_token": "authflowstate_ABC123DE45FG1",
    "type": "signup",
    "name": "default",
    "action": {
      "type": "verify",
      "data": {
        "channel": "email",
        "otp_form": "code",
        "masked_claim_value": "john******@example.com",
        "code_length": 6,
        "can_resend_at": "2023-09-21T00:00:00+08:00",
        "can_check": false,
        "failed_attempt_rate_limit_exceeded": false
      }
    }
  }
}
```

If `otp_form` is `code`, an OTP will be sent to the end user at `masked_claim_value`.

To resend OTP, send another request in a new step with the following input:

```json
"state_token": "authflowstate_ABC123DE45FG1",
"input": {
  "resend": true
}
```

Next, submit the OTP sent to the user using the following input in your request:

```json
"state_token": "authflowstate_XYZ987AA44Y9",
"input": {
  "code": "000000"
}
```

Or, you'll get the following response if the OTP form is a link:

```json
{
  "result": {
    "state_token": "authflowstate_IJK567UU1KL889J",
    "type": "signup",
    "name": "default",
    "action": {
      "type": "verify",
      "data": {
        "channel": "email",
        "otp_form": "link",
        "websocket_url": "wss://...",
        "masked_claim_value": "john******@example.com",
        "code_length": 32,
        "can_resend_at": "2023-09-21T00:00:00+08:00",
        "can_check": false,
        "failed_attempt_rate_limit_exceeded": false
      }
    }
  }
}
```

* if `otp_form` is `link`, `can_check` initially is `false` and `websocket_url` will be present in `data`. You can connect to a WebSocket with this URL to listen for the event of the link being approved.
* The verification link will be sent to the end user via the email in `masked_claim_value`. When the user clicks on the link,  the default approval page will open.&#x20;
* A WebSocket message of a JSON object `{"type": "refresh"}` is sent once the user approves the link. Upon receiving the message, you can [retrieve a state again](authentication-flow-api.md#id-5.1-retrieve-a-state-again).
* The retrieved state should have `can_check=true`.&#x20;

Now you can pass this input to check if the link has been approved:

```json
"state_token": "authflowstate_IJK567UU1KL889J",
"input": {
  "check": true
}
```

### 2.6 action.type: change\_password

The response that contains this action looks like this:

```json
{
  "result": {
    "state_token": "authflowstate_MNO876HH96SFF6",
    "type": "login",
    "name": "default",
    "action": {
      "type": "change_password",
      "data": {
        "password_policy": {
          "minimum_length": 8,
          "alphabet_required": true,
          "digit_required": true
        }
      }
    }
  }
}
```

To submit a new password send the following input in your request:

```json
{
  "new_password": "a.new.password.that.meet.the.password.policy"
}
```

### 2.7 action.type: reset\_password

The following is a sample of a response that contains this action:

```json
{
  "result": {
    "state_token": "authflowstate_5R6NM7HGGKV64538R0QEGY9RQBDM4PZD",
    "type": "signup",
    "name": "default",
    "action": {
      "type": "reset_password",
      "data": {
        "password_policy": {
          "minimum_length": 8,
          "digit_required": true,
          "history": {
            "enabled": false
          }
        }
      }
    }
  }
}
```

To complete the password reset process pass the following input:

```json
{
  "new_password": "a.new.password.that.meet.the.password.policy"
}
```

### 2.8 Finish Response

This is a special kind of response that is returned at the end of the authentication flow.

**Example:**

```json
{
  "result": {
    "state_token": "authflowstate_XYZ01234EF56GH8A0KFC998K",
    "type": "login",
    "name": "default",
    "action": {
      "type": "finished",
      "data": {
        "finish_redirect_uri": "https://myapp.authgear.cloud/..."
      }
    }
  }
}
```

* The value for `action.type` for the finish response is "finished" as this is the end of the authentication flow.
* `action.data.finish_redirect_uri` contains a URL that can be used to hand the authentication process back to Authgear so that it can redirect the user back to the client application. You can hand the authentication process back to Authgear by simply initiating a redirect to the URL in `action.data.finish_redirect_uri` from your custom authentication UI. See our example of [using the finish\_redirect\_uri to complete a signup flow](https://docs.authgear.com/how-to-guide/custom-ui/implement-authentication-flow-api-using-express#step-7-complete-the-signup-flow).

## 3.0 Input: Identifications

In this section, we'll cover all the supported identification options and how to pass them as input.

### 3.1 identification: email

Presence in response body:

```json
{
  "identification": "email"
}
```

Usage in input:

```json
{
  "identification": "email",
  "login_id": "user@example.com"
}
```

### 3.2 identification: phone

Presence in response body:

```json
{
  "identification": "phone"
}
```

Usage in input

```json
{
  "identification": "phone",
  "login_id": "+852980005432"
}
```

**Note**: The phone number MUST BE in E.164 format without any separators or spaces.

### 3.3 identification: username

Presence in response body:

```
{
  "identification": "username"
}
```

Usage in input:

```json
{
  "identification": "username",
  "login_id": "johndoe"
}
```

### 3.4 identification: oauth

Presence in response:

```
{
  "identification": "oauth",
  "provider_type": "google",
  "alias": "google"
}
```

* `provider_type`: The expected value here is a keyword for a specific OAuth provider. Possible values are:
  * `google`
  * `facebook`
  * `github`
  * `linkedin`
  * `azureadv2`
  * `azureadb2c`
  * `adfs`
  * `apple`
  * `wechat`
* `alias`: The identifier of the OAuth provider. You pass this in the input.

Usage in input:

```json
{
  "identification": "oauth",
  "alias": "google",
  "redirect_uri": "<https://example.com/oauth/redirect/google>"
}
```

The response to an Authentication Flow API request with the above input should look like this:

```json
{
    "result": {
        "state_token": "authflowstate_DVYQP8QXHDXYBNXGPPQZHM0YG3RC4SMH",
        "type": "signup",
        "name": "default",
        "action": {
            "type": "identify",
            "identification": "oauth",
            "data": {
                "alias": "google",
                "oauth_provider_type": "google",
                "oauth_authorization_url": "https://accounts.google.com/o/oauth2/v2/auth?client_id=850158775140-cuov06p5ru3kq27n4bfvrlp3fb7vrpgd.apps.googleusercontent.com&prompt=select_account&redirect_uri=http%3A%2F%2Flocalhost%3A3000&response_type=code&scope=openid+profile+email"
            }
        }
    }
}
```

In order to continue with the flow, redirect the user to the OAuth provider's authorization page using the link in `action.data.oauth_authorization_url`.

Once the user successfully grants your application authorization on the OAuth provider's site, they will get an authorization code. This code is usually returned by the provider in your OAuth redirect URL as a `code` query parameter.

Next, send the authorization code in your next HTTP request to the Authentication Flow API in your input like this:

```
{
  "code": "<AUTHORIZATION CODE FROM OAUTH PROVIDER>"
}
```

In some cases, the user may be unable to grant authorization successfully. In such situations, you can continue your authentication flow more gracefully by sending the following error-handling input in your next HTTP request instead:

```json
{
  "error": "{{ error }}",
  "error_description": "{{ error_description }}",
  "error_uri": "{{ error_uri }}"
}
```

`error_description` and `error_uri` are optional.

## 4.0 Input: Authentications

Here we'll cover all the supported authentication methods (authenticators) and how to pass them as input.

### 4.1 authentication: primary\_password

Presence in response:

```json
{
  "authentication": "primary_password",
  "password_policy": {
    "minimum_length": 8
  }
}
```

The value of `password_policy` is an object that contains all the requirements for a password. The following are all the possible attributes for the `password_policy` object:

```json
{
  "minimum_length": 8,
  "uppercase_required": true,
  "lowercase_required": true,
  "alphabet_required": true,
  "digit_required": true,
  "symbol_required": true,
  "minimum_zxcvbn_score": 4
}
```

Usage in input (for login flow):

```json
{
  "authentication": "primary_password",
  "password": "some.very.secure.password"
}
```

Usage in input (for signup flow)

```json
{
  "authentication": "primary_password",
  "new_password": "some.very.secure.password"
}
```

### 4.2 authentication: primary\_oob\_otp\_email

Presence in response (for login flow):

```json
{
  "authentication": "primary_oob_otp_email",
  "otp_form": "code",
  "masked_display_name": "john****@example.com",
  "channels": ["email"]
}
```

In the above response, `otp_form` tells you what kind of OTP will be sent. `masked_display_name` tells you what email address the OTP will be sent to. `channels` tells you the available channels you must choose from.

Presence in response (signup flow):

```json
{
  "authentication": "primary_oob_otp_email"
}
```

Usage in input (for login flow):

```json
{
  "authentication": "primary_oob_otp_email",
  "index": 1,
  "channel": "email"
}
```

To reference this authentication, use its index in `options` array. The next step for using primary\_oob\_otp\_email in your signup flow will be verifying the OTP sent to the user's email.

You can verify the OTP by sending the OTP in the input of the next request for your login flow like this:

```json
"input": {
        "code": "<OTP CODE SENT TO USER VIA EMAIL>"
    }
```

Usage in input (for signup flow)

```json
{
  "authentication": "primary_oob_otp_email"
}
```

### 4.3 authentication: primary\_oob\_otp\_sms

Presence in response (for login flow):

```json
{
  "authentication": "primary_oob_otp_sms",
  "otp_form": "code",
  "masked_display_name": "+8529876****",
  "channels": ["sms", "whatsapp"]
}
```

`otp_form` tells you what kind of OTP will be sent. `masked_display_name` tells you what phone number the OTP will be sent to. `channels` tells you the available channels you must choose from.

Presence in response (for signup flow):

```json
{
  "authentication": "primary_oob_otp_sms"
}
```

Usage in input (for login flow):

```json
{
  "authentication": "primary_oob_otp_sms",
  "index": 2,
  "channel": "sms"
}
```

To reference this authentication, use its index in `options` array.

The rest of the steps for implementing `primary_oob_otp_sms` in a login flow is similar to [primary\_oob\_otp\_email](authentication-flow-api.md#id-4.2-authentication-primary\_oob\_otp\_email). The major difference is the change in the `channel` from `email` to `sms`.

Usage in input(for signup flow):

```json
{
  "authentication": "primary_oob_otp_sms"
}
```

### 4.4 authentication: secondary\_password

Presence in response:

```json
{
  "authentication": "secondary_password",
  "password_policy": {
    "minimum_length": 8
  }
}
```

Usage in input (for login flow):

```json
{
  "authentication": "secondary_password",
  "password": "some.very.secure.password"
}
```

Usage in input (for signup flow):

```json
{
  "authentication": "secondary_password",
  "new_password": "some.very.secure.password"
}
```

### 4.5 authentication: secondary\_oob\_otp\_email

Presence in response:

```json
{
  "authentication": "secondary_oob_otp_email"
}
```

Usage in input:

```json
{
  "authentication": "secondary_oob_otp_email",
  "target": "user@example.com"
}
```

**Note**: `target` can be different (and is usually different) from the email address the end-user uses to sign in.

### 4.6 authentication: secondary\_oob\_otp\_sms

Presence in response:

```json
{
  "authentication": "secondary_oob_otp_sms"
}
```

Usage in input:

```json
{
  "authentication": "secondary_oob_otp_sms",
  "target": "+85298000032"
}
```

**Note**: `target` MUST BE in E.164 format without any separators or spaces. It can be different (and is usually different) from the phone number the end-user uses to sign in.

### 4.7 authentication: secondary\_totp

Presence in response:

```json
{
  "authentication": "secondary_totp"
}
```

Usage in input (for login flow)

```json
{
  "authentication": "secondary_totp",
  "code": "<TOTP CODE>"
}
```

Usage in input (for signup flow):

```json
{
  "authentication": "secondary_totp"
}
```

Sending  `"authentication": "secondary_totp"` in the input for your signup flow request will return a response that looks like this:

```json
{
    "result": {
        "state_token": "authflowstate_PS3KDB0DZQQ3QZRSM40XWMJPK9NQXZYQ",
        "type": "signup",
        "name": "default",
        "action": {
            "type": "create_authenticator",
            "authentication": "secondary_totp",
            "data": {
                "secret": "ABCDEFGHIJKLM9NOP08R2M7APSXYZ",
                "otpauth_uri": "otpauth://totp/pius1www2@example.com?algorithm=SHA1&digits=6&issuer=https%3A%2F%2Fyour-app.authgear.cloud&period=30&secret=ABCDEFGHIJKLM9NOP08R2M7APSXYZ"
            }
        }
    }
}
```

You can encode the value of `action.data.otpauth_uri` into a QR code that users can scan in an Authenticator app. Or display the value of `action.data.secret`so they can manually enter the secret in the Authenticator app.

To complete the TOTP authentication flow, send the code generated from the Authenticator app in your next request like this:

```json
{
  "code": "<CODE FROM AUTHENTICATOR APP>"
}
```

## 5.0 FAQ

### 5.1 Retrieve a state again

You can use the state token to retrieve more details about an existing state. You can do so by making the following HTTP request:

```http
POST /api/v1/authentication_flows/states
Content-Type: application/json

{
  "state_token": "{{ state_token }}"
}
```

Typically you do not need this because the state is returned after creation or after input was passed.

## 6.0 Error Handling

In this section, we'll cover how the Authentication Flow API handles different kinds of errors.

The Authentication Flow API returns an HTTP status code within the 400 range when a request is not successful. Or, an error with HTTP status code when there's an internal error in the server.

> Some fields may be absent in the error response from the Authentication Flow API depending on the type of error that occurred. Hence, we recommend doing checks for such fields in your code before using them in your application logic.

### 6.1 The error object.

The main HTTP response from the Authentication Flow API may include an `error` field when an `error` occurs in the authentication flow. The value of the `error` field is the error object. The `error` object includes the following fields:

* `name`: this is a short tag that qualifies a set of related errors.
* `reason`: The value of the reason field can be used to distinguish between different types of errors. It is more specific than the `name` field, and more reliable for usage in application logic than the `message` field as the value of the `message` field can change.
* `message`: the value of the `message` field is a string that contains more descriptive information about the error that has occurred.
* `code`: This is the HTTP status code associated with an error. Examples include 500, 400, 401, and 404.
* `info`: The info field may be absent in an error response. When it is present, the value is an object that contains more detailed information about the error that has occurred. Below are more details about some of the data in the `info` field.

#### Error.info

The `error.info` object may include these fields:

* `FlowType`: This field tells the type of authentication flow for which the error has occurred. For example, `login` and `signup` flows.
* `causes`: This is an array with more details about all the possible sources of the current error and hints on how to resolve the error.
* `cause`: The value of cause is an object with simple details about why the current error occurred.

To further understand the structure of the error response from the Authentication Flow API, lets consider some examples of common errors. We'll categorize the errors using the `error.reason`

### 6.2 ValidationFailed

The value of `error.reason` can be `ValidationFailed` when there's a missing required field. In the example below, the error occurred because the request to Authentication Flow API's `/api/v1/authentication_flows/states/input` endpoint is missing a required `input` or `batch_input` field.

```json
"error": {
        "name": "Invalid",
        "reason": "ValidationFailed",
        "message": "invalid request body",
        "code": 400,
        "info": {
            "causes": [
                {
                    "location": "",
                    "kind": "required",
                    "details": {
                        "actual": [
                            "state_token"
                        ],
                        "expected": [
                            "input"
                        ],
                        "missing": [
                            "input"
                        ]
                    }
                },
                {
                    "location": "",
                    "kind": "required",
                    "details": {
                        "actual": [
                            "state_token"
                        ],
                        "expected": [
                            "batch_input"
                        ],
                        "missing": [
                            "batch_input"
                        ]
                    }
                }
            ]
        }
    }
```

Another cause for a `ValidationFailed` error is an invalid format in the value of a required field or an invalid constant in a required field.&#x20;

An example of an invalid constant is setting the value of `identification` to anything outside the allowed values (`email`, `phone`, `oauth`, `username`). Also, using a constant for any feature that is not supported by your current login methods may throw the same error. You can always find more specific details about the cause of the error in the `error.info` object.&#x20;

The following example shows a `ValidationFailed` error for when a user enters a string that's not the correct format for an email address:

```json
"error": {
        "name": "Invalid",
        "reason": "ValidationFailed",
        "message": "invalid login ID",
        "code": 400,
        "info": {
            "FlowType": "signup",
            "causes": [
                {
                    "location": "/login_id",
                    "kind": "format",
                    "details": {
                        "format": "email"
                    }
                }
            ]
        }
    }
```

### 6.3 InvariantViolated

An `InvariantViolated` error may occur when the input for a signup flow contains an identity (email, phone, or username) that already exists for another user. You can find more specific details about the cause of the `InvariantViolated` error in the `error.info` object.

The following example shows an error response for a signup flow that attempts to register a new user with an email that already exists for another user:

```json
"error": {
        "name": "Invalid",
        "reason": "InvariantViolated",
        "message": "identity already exists",
        "code": 400,
        "info": {
            "FlowType": "signup",
            "IdentityTypeExisting": "login_id",
            "IdentityTypeIncoming": "login_id",
            "LoginIDTypeExisting": "email",
            "LoginIDTypeIncoming": "email",
            "cause": {
                "kind": "DuplicatedIdentity"
            }
        }
    }
```

### 6.4 PasswordPolicyViolated

When the value for a new password does not meet the minimum password requirement for your Authgear project, the Authentication Flow API will return an error with `PasswordPolicyViolated`. More details about how the password in the request fails to meet the requirement will be in the `error.info` object.

In the following example, the password in the request fails to meet the minimum length of 8 characters:

```json
"error": {
    "name": "Invalid",
    "reason": "PasswordPolicyViolated",
    "message": "password policy violated",
    "code": 400,
    "info": {
        "FlowType": "signup",
        "causes": [
            {
                "Name": "PasswordTooShort",
                "Info": {
                    "min_length": 8,
                    "pw_length": 4
                }
            }
        ]
    }
}
```

### 6.5 AuthenticationFlowNotFound

The value of  `error.reason` can be `AuthenticationFlowNotFound` when the `state_token` in the request is invalid or expired. This type of error does not include the `info` field.

```json
"error": {
        "name": "NotFound",
        "reason": "AuthenticationFlowNotFound",
        "message": "flow not found",
        "code": 404
    }
```

### 6.6 InvalidCredentials

The Authentication Flow API will return an `InvalidCredentials` error when an authentication fails because the input value for an authenticator is invalid ( e.g. a wrong password is entered for an existing user).

More details about the authenticator type can be found in the `error.info` object as shown below:

```json
"error": {
        "name": "Unauthorized",
        "reason": "InvalidCredentials",
        "message": "invalid credentials",
        "code": 401,
        "info": {
            "AuthenticationType": "password",
            "FlowType": "login"
        }
    }
```

### 6.7 UserNotFound

A `UserNotFound` error will occur when the input value for user identity (email, phone, username) does not exist in an Authgear project. In other words, no user was found for the email, phone number, or username.

More details about the error will be in the error.info object.

```json
"error": {
        "name": "NotFound",
        "reason": "UserNotFound",
        "message": "user not found",
        "code": 404,
        "info": {
            "FlowType": "login",
            "IdentityTypeIncoming": "login_id",
            "LoginIDTypeIncoming": ""
        }
    }
```

### 6.8 UnexpectedError

This is a rare error that may occur due to improper inputs such as poorly formatted JSON (e.g. including a trailing comma in JSON).

```json
"error": {
        "name": "InternalError",
        "reason": "UnexpectedError",
        "message": "unexpected error occurred",
        "code": 500
    }
```





