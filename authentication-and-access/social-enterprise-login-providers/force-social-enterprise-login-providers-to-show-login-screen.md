---
description: Use OIDC prompt parameter to force OAuth providers to show login screen.
---

# Force Social/Enterprise Login Providers to Show Login Screen

The `prompt="login"` parameter which is defined in the [OIDC spec](https://openid.net/specs/openid-connect-core-1\_0.html#AuthRequest) can prompt Social/Enterprise Login Providers to always show their login screen. As a result, you can use the `prompt="login"` parameter to allow users to switch accounts when their previous authentication session on the provider is still stored.&#x20;

In this post, you'll learn how to use `prompt: "login"` in Authgear SDKs to force OIDC providers to always show your users their login screen. You can also use `prompt: "login"` to allow users to switch accounts with a Social/Enterprise Login provider on the same device (or browser).

### How to Force Social/Enterprise Login Providers to Show Login Page

Authgear SDKs have a `prompt` parameter that you can set in your application. The value of the prompt parameter will be passed to the Social/Enterprise Login provider. Hence, if you set `prompt: "login"` in the SDK, your Social/Enterprise Login provider will receive a `prompt="login"` parameter. The following code examples show how to use the prompt parameter in Authgear SDKs.

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

