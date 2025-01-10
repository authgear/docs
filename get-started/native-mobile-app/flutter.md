---
description: How to integrate with a Flutter app
---

# Flutter SDK

This guide provides instructions on integrating Authgear with a Flutter app. Supported platforms include:

* Flutter 2.5.0 or higher

## Setup Application in Authgear

Signup for an Authgear Portal account in [https://portal.authgear.com/](https://portal.authgear.com/). Or you can use your self-deployed Authgear.

From the Project listing, create a new Project or select an existing Project. After that, we will need to create an application in the project.

{% tabs %}
{% tab title="Portal" %}
**Step 1: Create an application in the Portal**

1. Go to **Applications** on the left menu bar.
2. You will see the **"New Application"** page or Click **⊕Add Application** in the top tool bar.
3. Input the name of your application and select **Native App** as the application type. Click "Save".
4. You will see a list of guides that can help you for setting up, then click "Next".

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption><p>Create Application</p></figcaption></figure>

**Step 2: Configure the application**

1. In your IDE, define a custom URI scheme that the users will be redirected back to your app after they have authenticated with Authgear, e.g. `com.example.myapp://host/path`.\[^1]
2. Head back to Authgear Portal, fill in the Redirect URI that you have defined in the previous steps.
3. Click "Save" in the top tool bar and keep the **Client ID**. You can also obtain it again from the Applications list later.

![Edit an application](../../.gitbook/assets/edit-application-app.png)
{% endtab %}

{% tab title="authgear.yaml (self-deployed)" %}
```yaml
oauth:
  clients:
    - name: your_app_name
      client_id: a_random_generated_string
      redirect_uris:
        - "com.myapp.example://host/path"
      grant_types:
        - authorization_code
        - refresh_token
      response_types:
        - code
        - none
```
{% endtab %}
{% endtabs %}

## Create a Flutter app

Follow the documentation of Flutter to see how you can create a new Flutter app.

```bash
flutter create myapp
cd myapp
```

## Install the SDK

```bash
flutter pub add flutter_authgear
```

## Platform Integration

To finish the integration, setup the app to handle the redirectURI specified in the application. This part requires platform specific integration.

This declares the URL schemes supported by your app, so the device can redirect the user to the app after authentication using the redirect URI.

### Android

Add the following `<activity>` entry to the `AndroidManifest.xml` of your app. The intent system would dispatch the redirect URI to `OAuthRedirectActivity` and the SDK would handle the rest.

```markup
<!-- Your application configuration. Omitted here for brevity -->
<application>
  <!-- Other activities or entries -->

  <!-- Add the following activity -->
  <!-- android:exported="true" is required -->
  <!-- See https://developer.android.com/about/versions/12/behavior-changes-12#exported -->
  <activity android:name="com.authgear.flutter.OAuthRedirectActivity"
            android:exported="true"
            android:launchMode="singleTask">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <!-- Configure data to be the exact redirect URI your app uses. -->
                <!-- Here, we are using com.authgear.example://host/path as configured in authgear.yaml. -->
                <!-- NOTE: The redirectURI supplied in AuthenticateOptions *has* to match as well -->
                <data android:scheme="com.example.myapp"
                    android:host="host"
                    android:pathPrefix="/path"/>
            </intent-filter>
  </activity>
</application>
```

#### Targeting API level 30 or above (Android 11 or above)

If your Android app is targeting API level 30 or above (Android 11 or above), you need to add a `queries` section to `AndroidManifest.xml`.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
  <!-- Other elements such <application> -->
  <queries>
    <intent>
      <action android:name="android.support.customtabs.action.CustomTabsService" />
    </intent>
  </queries>
</manifest>
```

### iOS

#### Declare URL Handling in Info.plist

In `Info.plist`, add the matching redirect URI by adding the key `CFBundleURLTypes` and the values inside `<dict>` as shown as the following example.

```markup
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
      <!-- Other entries -->
      <key>CFBundleURLTypes</key>
      <array>
              <dict>
                      <key>CFBundleTypeRole</key>
                      <string>Editor</string>
                      <key>CFBundleURLSchemes</key>
                      <array>
                              <string>com.example.myapp</string>
                              <!-- Put the redirect URI your app uses here. -->
                      </array>
              </dict>
      </array>
</dict>
</plist>
```

## Try authenticate

Add this code to your app. This snippet configures authgear to connect to an authgear server deployed at `endpoint` with the client you have just setup via `clientID`, opens a browser for authentication, and then upon success redirects to the app via the `redirectURI` specified.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_authgear/flutter_authgear.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatefulWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  State<MyApp> createState() => _MyAppState();
}


class _MyAppState extends State<MyApp> {
  late Authgear _authgear;

  @override
  void initState() {
    super.initState();
    _init();
  }

  Future<void> _init() async {
    _authgear = Authgear(endpoint: "ENDPOINT", clientID: "CLIENT_ID");
    await _authgear.configure();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: "MyApp",
      home: Scaffold(
        appBar: AppBar(title: const Text("MyApp")),
        body: TextButton(
          child: Text("Authenticate"),
          onPressed: _onPressedAuthenticate,
        ),
      ),
    );
  }

  Future<void> _onPressedAuthenticate() async {
    try {
      await _authgear.authenticate(redirectURI: "REDIRECT_URI");
    } on CancelException {
    }
  }
}
```

Now, your user is logged in!

## Get the Logged In State

When you start launching the application. You may want to know if the user has logged in. (e.g. Show users the login page if they haven't logged in). The `sessionState` reflects the user logged in state in the SDK local state. That means even the `sessionState` is `SessionState.authenticated`, the session may be invalid if it is revoked remotely. After initializing the Authgear SDK, call `fetchUserInfo` to update the `sessionState` as soon as it is proper to do so.

```dart
// After authgear.configure, it only reflect SDK local state.
// value can be SessionState.noSession or SessionState.authenticated
SessionState state = authgear.sessionState;

UserInfo? userInfo;
try {
  userInfo = await authgear.getUserInfo();
  // read the userInfo if needed
} catch (e) {
  // failed to fetch user info
  // the refresh token maybe expired or revoked
}
// sessionState is now up to date
// it will change to SessionState.noSession if the session is invalid
state = authgear.sessionState;
```

The value of `sessionState` can be `SessionState.unknown`, `SessionState.noSession` or `SessionState.authenticated`. Initially, the `sessionState` is `SessionState.unknown`. After a call to `authgear.configure`, the session state would become `SessionState.authenticated` if a previous session was found, or `SessionState.noSession` if such session was not found.

## Fetching User Info

In some cases, you may need to obtain current user info through the SDK. (e.g. Display email address in the UI). Use the `fetchUserInfo` function to obtain the user info, see [example](../../how-to-guide/user-profiles/user-profile.md#userinfo-endpoint).

## Using the Access Token in HTTP Requests

To include the access token to the HTTP requests to your application server, use `wrapHttpClient`.

The wrapped client will include the Authorization header in every HTTP request, and refresh access token automatically.

```dart
final originalClient = ...
final client = authgear.wrapHttpClient(originalClient);
```

## Logout

To log out the user from the current app session, you need to invoke the`logout`function.

```dart
await authgear.logout();
```

## Next steps

To protect your application server from unauthorized access. You will need to **integrate your backend with Authgear**.

{% content-ref url="../backend-api/" %}
[backend-api](../backend-api/)
{% endcontent-ref %}

## Flutter SDK Reference

For detailed documentation on the Flutter SDK, visit [Flutter SDK Reference](https://authgear.github.io/authgear-sdk-flutter/)

### Footnote

\[^1]: For futher instruction on setting up custom URI scheme in Flutter, see [https://docs.flutter.dev/development/ui/navigation/deep-linking](https://docs.flutter.dev/development/ui/navigation/deep-linking)

\[^2]: For more explanation on JWT, see [https://en.wikipedia.org/wiki/JSON\_Web\_Token](https://en.wikipedia.org/wiki/JSON_Web_Token)
