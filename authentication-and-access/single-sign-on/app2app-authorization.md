---
description: >-
  Perform faster authentication flow via another app installed on the same
  device.
---

# App2App Login

This may be familiar for users from UK, which many neobanks are using the app2app mechanism to authorize the money transfer from 1 bank app to another.

The App2App mechanism allows one app to authenticate the user using another apps connected to the auth server installed on the same device. This is achieved by universal links and the apps do not need to share the session via the system browser or the refresh tokens via the token storage.

{% hint style="info" %}
This is an Enterprise feature, please contact us for using the App2App flow in your project at [https://www.authgear.com/talk-with-us](https://www.authgear.com/talk-with-us)
{% endhint %}

Please note that this is not the Single Sign-on feature, if your are offering multiple apps under the same brand and wish the users to use a shared login session among their apps in the device, you may want to use [Single Sign-on](./) instead. App2app should be used when:

* The session cannot be shared via the browser cookies
* The session cannot be shared via a common token storage

{% embed url="https://youtu.be/Q6FPiQm56xE" %}
Demo of the App2App login flow
{% endembed %}

### Mechanism

An app can start the authentication flow by opening a link to another app, instead of using the authorization endpoint. The app which handles the link should validate the authentication request, then could return a valid authorization code. The valid code is then transferred to the original app using universal link. The initiating app can use that authorization code to perform code exchange for tokens with Authgear.

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption><p>App2App flow in iOS</p></figcaption></figure>

A detailed explanation on the technology can be found in [this specification](https://github.com/authgear/authgear-server/blob/main/docs/specs/app2app.md).

### Setting it up in Authgear

{% hint style="info" %}
This is an Enterprise feature, please contact us for enabling the App2App flow in your project at [https://www.authgear.com/talk-with-us](https://www.authgear.com/talk-with-us)
{% endhint %}

#### Configuration in the authorizing app

1. Go to the Application detail page of the authorizing app, i.e. the app which handles the app2app authentication requests.
2. Scroll to the bottom and you will see the App2App config panel.
3. Select Enable App2App login for this Application"\
   <img src="../../.gitbook/assets/image (33).png" alt="" data-size="original">
4. Migration mode offers a less secure mechanism which helps older user sessions to participate in App2App. DO NOT enable it unless there is migration problem.

#### Configuration in the initiating app

1. Go to the Application detail page of the initiating app, i.e. the app which initiates the app2app authentication requests.
2. In the redirect URIs, a universal link that's capable of opening this app should be set.

### Setting up the apps

1. Define and set up the universal links for both apps, for example:
   1. `https://a.example.com/authorize` should open the authorizing app (App A)
   2. `https://b.example.com/redirect` should open the initiating app (App B)
2. In App B, call `startApp2AppAuthentication(options: App2AppAuthenticateOptions)` to initiate the app2app login
   * `App2AppAuthenticateOptions.authorizationEndpoint` should be an url of an universal link pointing to App A, i.e. `https://a.example.com/authorize`
   * `App2AppAuthenticateOptions.redirectUri` should be an URI for the authorizing app to return the authentication result. It must be an universal link which opens the current app. i.e. `https://b.example.com/redirect`
3. In App A, upon receiving the app2app login request
   * Call `parseApp2AppAuthenticationRequest(url: URL): App2AppAuthenticateRequest?`
   * The result will be `null` if the url is not a valid app2app request.
4. You can approve or reject the app2app request in App A
   * Approve: `approveApp2AppAuthenticationRequest(request: App2AppAuthenticateRequest)`
     * Approves an app2app request returning the result through the redirect URI.
     * `request` should be the return value of `parseApp2AppAuthenticationRequest`.
     * This method must be called when then SDK session state is `AUTHENTICATED`, and the current session supported app2app authentication by providing a `device_key`, or else an error will be thrown.
   * Reject: `rejectApp2AppAuthenticationRequest(request: App2AppAuthenticateRequest, error: Error)`
     * Rejects an app2app request, returning an error through the redirect URI.
     * `request` should be the return value of `parseApp2AppAuthenticationRequest`.
     * `error` is the reason to reject the request.
5. When it's back to App B, call `handleApp2AppAuthenticationResult(url: URL)`
   * This method should be called by the app which initiate the app2app authentication flow, and when received the result through the universal link, `url` should be the URL of the universal link received.
