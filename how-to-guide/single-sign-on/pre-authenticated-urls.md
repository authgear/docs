---
description: >-
  Use the pre-authenticated URLs feature to open a website from a native app in
  an authenticated state.
---

# Pre-authenticated URLs

Pre-authenticated URLs is a feature that enables single sign-on (SSO)  from a mobile application to a website. It allows users who are authenticated on a mobile application to open a website in an authenticated state.

An example use case for a pre-authenticated URL is opening a web application in a WebView.

### Prerequisites

&#x20;To use pre-authenticated URLs, you must have the following:

* A native app using Authgear as authentication
* A web application using the same Authgear project as authentication

## How to Implement Pre-authentication URLs in your application

### Step 1: Enable SSO & Pre-authenticated URLs in Native Client App

First, ensure your mobile application uses an Authgear application with the **Native App**. Enable both SSO and "preAuthenticatedURL" to allow pre-authenticated URLs to work.

{% tabs %}
{% tab title="Ionic" %}
```javascript
authgear
  .configure({
    clientID: '<CLIENT_ID>',
    endpoint: '<AUTHGEAR_PROJECT_URL>',
    isSSOEnabled: true,
    preAuthenticatedURLEnabled: true
  })
```
{% endtab %}
{% endtabs %}

### Step 2: Add Allowed Origin to Web App Client

Next, add an allowed origin to the web application client in Authgear. Navigate to **Applications** in the Authgear Portal, select the web application client, and scroll to the Allowed Origins section. Then, add the origin you wish to use for Pre-authentication URLs. Note that the origin should be of the format "protocol (scheme) + domain + port". For example, if the mobile application wants to open `https://www.mywebapp.com/home?key=value`, the origin must be `https://www.mywebapp.com`.

### Step 3: Generate Pre-Authenticated URL

The Pre-Authenticated URL is a link that the Authgear SDK can generate for a mobile client that has the Pre-Authenticated URLs feature enabled. Your mobile application can open the Pre-Authenticated URL in a web view for users to start browsing the origin in an authenticated state.

To generate the Pre-Authenticated URL, call the `makePreAuthenticatedURL()` method of the Authgear SDK as shown below:

{% tabs %}
{% tab title="Ionic" %}
```javascript
const url = await authgear.makePreAuthenticatedURL({
    webApplicationClientID: "YOUR_WEB_APP_CLIENT_ID", // Replace with you web app client id
    webApplicationURI: "YOUR_WEB_APP_URI", // Replace with you web app uri
  });
```
{% endtab %}
{% endtabs %}

The `makePreAuthenticatedURL()` method accepts an object as a parameter. Inside the object, you should provide your web application's client ID and web app URI.

### Step 4: Open Pre-Authenticated URL in a WebView

After the `makePreAuthenticatedURL()` return the URL, your mobile application should open the URL in a WebView. From there, users should be able to continue their current authenticated session (from the mobile app) on the web application.

{% tabs %}
{% tab title="Ionic" %}
The following code sample shows how to open the pre-authenticated URL using the `Browser.open()` method in Ionic.

<pre class="language-javascript"><code class="lang-javascript"><strong>Browser.open({ url: url }).catch(err =>
</strong>      console.error("Couldn't load page", err),
);
</code></pre>
{% endtab %}

{% tab title="React Native" %}
The following code sample shows how to open the pre-authenticated URL using the `Linking.openURL()` method of React Native.

<pre><code><strong>Linking.openURL(url).catch(err =>
</strong>      console.error("Couldn't load page", err),
);
</code></pre>
{% endtab %}
{% endtabs %}

### Step 5: Get authenticated state in the web application

The pre-authenticated URL is opened in the browser via the native app. In the web application, trigger authentication with the injected SSO session and get the authenticated state.

{% tabs %}
{% tab title="Web" %}
In the web application, enable SSO to allow pre-authenticated URLs to work. You can initialize the SDK as following

```typescript
import authgear, { PromptOption } from "@authgear/web";

authgear.configure({
    endpoint: "AUTHGEAR_ENDPOINT",
    clientID: "CLIENT_ID",
    sessionType: "refresh_token",
    isSSOEnabled: true
});
```

And in the web application URI, trigger authentication as following. Note here `prompt: PromptOption.None` is used to skip the SSO continue screen. &#x20;

```typescript
import authgear, { PromptOption } from "@authgear/web";

authgear.startAuthentication({
  redirectURI: import.meta.env.VITE_AUTHGEAR_REDIRECT_URL,
  prompt: PromptOption.None // use "None" to skip the continue screen
})
.then(
  () => {
    // started authentication, user should be redirected to Authgear
   },
  (err) => {
    // failed to start authorization
  }
);
```

In a normal login flow, for example the user browses the web page in the browser rather than from a link in the native app, the prompt should not be used because it will hinder the user from opening the login page. Only use this prompt when an SSO session is surely set in the browser, for instance in conjunction with this Pre-authentication URL feature.
{% endtab %}
{% endtabs %}
