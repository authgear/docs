---
description: >-
  Force Authgear to always show login page even if the user have already logged
  in.
---

# Force Authgear to Show Login Page

When user login / signup to Authgear, it usually starts with your application making a request to the authorization endpoint, which leads to a login or signup screen.

If the user is already signed in on the browser, the Single Sign On feature will show a "Continue Screen" instead as follows.

<figure><img src="../../.gitbook/assets/authgear-continue-screen.png" alt=""><figcaption></figcaption></figure>

If your application do not want to utilize the Single Sign On feature, and always show the login / sign up screen instead, you can force Authgear to show login page by using `prompt="login"`  at the `authorize` endpoint.

### How to Force Authgear to Show Login Page

The `prompt="login"` parameter which is defined in the [OIDC spec](https://openid.net/specs/openid-connect-core-1\_0.html#AuthRequest) can force AuthUI to show the login page. Authgear SDKs have a `prompt` parameter that can be used to set prompt="login". Once the `prompt` parameter is set to `login` Authgear will always show the login screen when your application calls the SDK's authenticate method.

The following code shows how to set `prompt: "login"` in Authgear SDKs:

{% tabs %}
{% tab title="JavaScript" %}
```javascript
authgear
  .startAuthentication({
    redirectURI: "<AUTHGEAR_REDIRECT_URI>",
    prompt: PromptOption.Login,
  })
```
{% endtab %}

{% tab title="React Native" %}
```typescript
authgear
  .authenticate({
    redirectURI: 'com.reactnativeauth://host/path',
    prompt: PromptOption.Login,
  })
```
{% endtab %}

{% tab title="Android" %}
```java
AuthenticateOptions options = new AuthenticateOptions("<AUTHGEAR_REDIRECT_URI>");
List<PromptOption> promptOptions = Arrays.asList(PromptOption.LOGIN);
options.setPrompt(promptOptions);
mAuthgear.authenticate(options, new OnAuthenticateListener() {
    @Override
    public void onAuthenticated(@Nullable UserInfo userInfo) {
        
    }

    @Override
    public void onAuthenticationFailed(@NonNull Throwable throwable) {
        Log.d(TAG, throwable.toString());
    }
});
```
{% endtab %}

{% tab title="iOS" %}
```swift
authgear?.authenticate(
    redirectURI: "<AUTHGEAR_REDIRECT_URI>",
    prompt: "login"
)
```
{% endtab %}

{% tab title="Flutter" %}
```dart
_authgear.authenticate(
        redirectURI: "<AUTHGEAR_REDIRECT_URI>",
        prompt: "login",
      );
```
{% endtab %}
{% endtabs %}

