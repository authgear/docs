# Force authentication on app launch

If your mobile app has security requirements similar to that of mobile banking applications, you may want the end-users to authenticate themselves every time they use your app.

By default, the SDK stores the refresh token in a persistent storage specific to your app. The end-user signs in once and their session lasts for a long period, even if they quit the app.

You can alter this behavior by switching to a transient storage by setting the `tokenStorage` option to `TransientTokenStorage()` when configuring the SDK. The refresh token will be removed after the app is cleared, therefore authentication will be required on every app launch.

{% tabs %}
{% tab title="React Native" %}
```typescript
import authgear, { TransientTokenStorage } from "@authgear/react-native";

authgear.configure({
    clientID: CLIENT_ID,
    endpoint: ENDPOINT,
    tokenStorage: new TransientTokenStorage(),
});
```
{% endtab %}

{% tab title="Flutter" %}
```dart
final authgear = Authgear(
    clientID: CLIENT_ID,
    endpoint: ENDPOINT,
    tokenStorage: TransientTokenStorage(),
);
```
{% endtab %}

{% tab title="Xamarin" %}
```csharp
var authgearOptions = new AuthgearOptions
{
    ClientId = CLIENT_ID,
    AuthgearEndpoint = ENDPOINT,
    TokenStorage: new TransientTokenStorage(),
};
#if __ANDROID__
var authgear = new AuthgearSdk(GetActivity().ApplicationContext, authgearOptions);
#else
#if __IOS__
var authgear = new AuthgearSdk(UIKit.UIApplication.SharedApplication, authgearOptions);
#endif
#endif
```
{% endtab %}

{% tab title="iOS" %}
```swift
Authgear(
    clientId: CLIENT_ID,
    endpoint: ENDPOINT,
    tokenStorage: TransientTokenStorage()
)
```
{% endtab %}

{% tab title="Android" %}
```java
new Authgear(
    application,
    CLIENT_ID,
    ENDPOINT,
    new TransientTokenStorage() //tokenStorage
);
```
{% endtab %}
{% endtabs %}
