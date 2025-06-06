---
description: >-
  How to make authorized request to your application server after login with
  Authgear
---

# Use SDK to make authorized API calls to backend

## Using Authgear SDK to call your application server

In this section, we are going to explain how to make an authorized request to your backend API by using Authgear SDK. Authgear SDKs make it easy to refresh access token if needed and maintain session state.

If you are using **Cookie-based authentication** in your web application, you can skip this section as session cookies are handled by the browser automatically.

## Overview

1. To determine which user is calling your server, you will need to include the Authorization header in every request that send to your application server.
2. On your Backend/API, you will need to set up [Backend integration](broken-reference). Authgear will help you to handle the Authorization header to determine whether the incoming HTTP request is authenticated or not.

In the below section, we will explain how to set up SDK to for the purpose of making authorized API calls to your backend.

## SDK Setup

Configure the Authgear SDK with the Authgear endpoint and client id. The SDK must be properly configured before use by calling `configure`. No network call will be triggered during `configure`.

{% tabs %}
{% tab title="Javascript" %}
```javascript
authgear
    .configure({
        clientID: "<YOUR_APPLICATION_CLIENT_ID>",
        endpoint: "<YOUR_AUTHGEAR_ENDPOINT>",
    })
    .then(() => {
        // configured successfully
    })
    .catch((e) => {
        // failed to configured
    });
```
{% endtab %}

{% tab title="iOS" %}
```swift
let authgear = Authgear(clientId: clientId, endpoint: endpoint)
authgear.configure() { result in
    switch result {
    case .success():
        // configured successfully
    case let .failure(error):
        // failed to configured
    }
}
```
{% endtab %}

{% tab title="Android" %}
```java
ConfigureOptions configureOptions = new ConfigureOptions();
Authgear authgear = new Authgear(getApplication(), clientID, endpoint);
authgear.configure(configureOptions, new OnConfigureListener() {
    @Override
    public void onConfigured() {
        // configured successfully
    }

    @Override
    public void onConfigurationFailed(@NonNull Throwable throwable) {
        // failed to configured
    }
});
```
{% endtab %}

{% tab title="Xamarin" %}
```csharp
var authgearOptions = new AuthgearOptions
{
    ClientId = "<YOUR_APPLICATION_CLIENT_ID>",
    AuthgearEndpoint: "<YOUR_AUTHGEAR_ENDPOINT>",
};
#if __ANDROID__
var authgear = new AuthgearSdk(GetActivity().ApplicationContext, authgearOptions);
#else
#if __IOS__
var authgear = new AuthgearSdk(UIKit.UIApplication.SharedApplication, authgearOptions);
#endif
#endif
await authgear.ConfigureAsync();
```
{% endtab %}
{% endtabs %}

## UserInfo and Session State

`sessionState` reflect the user logged in state. However the session state is cached locally and only updated after each server call.&#x20;

Usually right after login/signup via `authorize`,You will call `fetchUserInfo` as soon as possible with `authgear.sessionState` became `AUTHENTICATED`

{% tabs %}
{% tab title="Javascript" %}
```javascript
// value can be NO_SESSION or AUTHENTICATED
// After authgear.configure, it only reflect SDK local state.
let sessionState = authgear.sessionState;

if (sessionState === "AUTHENTICATED") {
    authgear
        .fetchUserInfo()
        .then((userInfo) => {
            // sessionState is now up to date
            // read the userInfo if needed
        })
        .catch((e) => {
            // sessionState is now up to date
            // it will change to NO_SESSION if the session is invalid
        });
}
```
{% endtab %}

{% tab title="iOS" %}
```swift
// value can be .noSession or .authenticated.
// After authgear.configure, it only reflect SDK local state.
let sessionState = authgear.sessionState

// call fetchUserInfo to see if the session is valid
if authgear.sessionState == .authenticated {
    authgear.fetchUserInfo { userInfoResult in
        // sessionState is now up to date
        // it will change to .noSession if the session is invalid
        let sessionState = authgear.sessionState

        switch userInfoResult {
        case let .success(userInfo):
            // read the userInfo if needed
        case let .failure(error):
            // failed to fetch user info
            // the refresh token maybe expired or revoked
    }
}
```
{% endtab %}

{% tab title="Android" %}
```java
// value can be NO_SESSION or AUTHENTICATED
// After authgear.configure, it only reflect SDK local state.
SessionState sessionState = authgear.getSessionState();

if (sessionState == SessionState.AUTHENTICATED) {
    authgear.fetchUserInfo(new OnFetchUserInfoListener() {
        @Override
        public void onFetchedUserInfo(@NonNull UserInfo userInfo) {
            // sessionState is now up to date
            // read the userInfo if needed
        }

        @Override
        public void onFetchingUserInfoFailed(@NonNull Throwable throwable) {
            // sessionState is now up to date
            // it will change to NO_SESSION if the session is invalid
        }
    });
}
```
{% endtab %}

{% tab title="Xamarin" %}
```csharp
// value can be NoSession or Authenticated
// After Authgear.ConfigureAsync, it only reflects local state.
var sessionState = authgear.SessionState;

if (sessionState == SessionState.Authenticated)
{
    try
    {
        var userInfo = await authgear.FetchUserInfoAsync();
        // sessionState is now up to date
    }
    catch (Exception ex)
    {
        // sessionState is now up to date
        // it will change to NoSession if the session is invalid
    }
}
```
{% endtab %}
{% endtabs %}

## Makeing an API call

When you make a API call to Backend API, you will need to include the access token in the `Authorization` header. Access token is also short lived and need to be regularly rotated by Refresh token for security purpose. Authgear SDKs provide the following functions to simplify both steps.

### **The `fetch` function (JavaScript Only)**

**Javascript Only**. Authgear SDK provides `fetch` function for you to call your application server. The `fetch` function will include Authorization header in your application request, and handle refresh access token automatically. `authgear.fetch` implement [fetch](https://fetch.spec.whatwg.org/).&#x20;

If you are using another networking library, you will need to use `refreshAccessTokenIfNeeded().` and include the `Authorization` header yourself, as described in the next paragraph.

{% tabs %}
{% tab title="Javascript" %}
```javascript
authgear
    .fetch("YOUR_SERVER_URL")
    .then(response => response.json())
    .then(data => console.log(data));
```
{% endtab %}
{% endtabs %}

### The refreshAccessTokenIfNeeded function

You will need to include the Authorization header in your application request. Call `refreshAccessTokenIfNeeded` every time before using the access token, the function will check and make the network call only if the access token has expired. Include the access token into the Authorization header of your application request.

{% tabs %}
{% tab title="Javascript" %}
```java
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

{% tab title="iOS" %}
```swift
authgear.refreshAccessTokenIfNeeded() { result in
    switch result {
    case .success():
        // access token is ready to use
        // accessToken can be empty
        // it will be empty if user is not logged in or session is invalid

        // include Authorization header in your application request
        if let accessToken = authgear.accessToken {
            // example only, you can use your own networking library
            var urlRequest = URLRequest(url: "YOUR_SERVER_URL")
            urlRequest.setValue(
                "Bearer \(accessToken)", forHTTPHeaderField: "authorization")
            // ... continue making your request
        }
    case let .failure(error):
        // failed to refresh access token
        // the refresh token maybe expired or revoked
    }
}
```
{% endtab %}

{% tab title="Android" %}
```java
// Suppose we are preparing an http request in a background thread.

// Setting up the request, e.g. preparing a URLConnection

try {
    authgear.refreshAccessTokenIfNeededSync();
} catch (OauthException e) {
    // failed to refresh access token
    // the refresh token maybe expired or revoked
}
// access token is ready to use
// accessToken can be string or undefined
// it will be empty if user is not logged in or session is invalid
String accessToken = authgear.getAccessToken();
HashMap<String, String> headers = new HashMap<>();
headers.put("authorization", "Bearer " + accessToken);

// Submit the request with the headers...
```
{% endtab %}

{% tab title="Xamarin" %}
```csharp
try
{
    await authgear.RefreshAccessTokenIfNeededAsync();
}
catch (OauthException ex)
{
    // failed to refresh access token
    // the refresh token maybe expired or revoked
}
// access token is ready to use
// accessToken can be string or undefined
// it will be empty if user is not logged in or session is invalid
var accessToken = authgear.AccessToken;
var client = GetHttpClient();  // Get the re-used http client of your app, as per recommendation.
var httpRequestMessage = new HttpRequestMessage(myHttpMethod, myUrl);
httpRequestMessage.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
// Send the request with the headers...
```
{% endtab %}
{% endtabs %}

## Handle revoked sessions

If the session is revoked from the management portal, the client will call your Backend API with an invalid access token. Your application server can check that by looking at the [resolver headers](broken-reference).

For example, you application may return HTTP status code 401 for unauthorized requests. Depending on your application flow, you may want to show your user login page again or reset the SDK `sessionState` to `NO_SESSION` locally. To clear the `sessionState`, you can use `clearSessionState` function.

{% tabs %}
{% tab title="Javascript" %}
```javascript
// example only
// if your application server return HTTP status code 401 for unauthorized request
async function fetchAppServer() {
    var response = await authgear.fetch("YOUR_SERVER_URL");
    if (response.status === 401) {

        // if you want to clear the session state locally, call clearSessionState
        // `authgear.sessionState` will become `NO_SESSION` after calling
        await authgear.clearSessionState();
        throw new Error("user session invalid");
    }
    // ...
}
```
{% endtab %}

{% tab title="iOS" %}
```swift
// example only
// if your application server return HTTP status code 401 for unauthorized request
if let response = response as? HTTPURLResponse {
    if response.statusCode == 401 {

        // if you want to clear the session state locally, call clearSessionState
        // `authgear.sessionState` will become `.noSession` after calling
        authgear.clearSessionState { result in
            switch result {
            case .success():
                // clear SDK session state locally
                // `authgear.sessionState` becomes `.noSession`
            case let .failure(error):
                // failed to clear session state
            }
        }
    }
}
```
{% endtab %}

{% tab title="Android" %}
```java
// example only
// if your application server return HTTP status code 401 for unauthorized request
responseCode = httpConn.getResponseCode();
if (responseCode == HttpURLConnection.Unauthorized) {

    // if you want to clear the session state locally, call clearSessionState
    // `authgear.getSessionState()` will become `NO_SESSION` after calling
    authgear.clearSessionState();
}
```
{% endtab %}

{% tab title="Xamarin" %}
```csharp
// example only
// if your application server return HTTP status code 401 for unauthorized request
statusCode = httpResponseMessage.StatusCode;
if (statusCode == HttpStatusCode.Unauthorized)
{
    // if you want to clear the session state locally, call ClearSessionState
    // `authgear.SessionState` will become `NoSession` after calling
    authgear.ClearSessionState();
}
```
{% endtab %}
{% endtabs %}
