---
description: How to integrate with a Xamarin app
---

# Xamarin SDK

[![LLM | View as markdown](https://img.shields.io/badge/LLM-View%20as%20markdown-blue)](https://r.jina.ai/https://docs.authgear.com/get-started/native-mobile-app/xamarin)

This guide provides instructions on integrating Authgear with a Xamarin app. Supported packages include:

* Xamarin.Essentials 1.7.2 or higher
* Xamarin.Forms 5.0.0.2401 or higher

## Setup Application in Authgear

Signup for an Authgear Portal account in [https://portal.authgear.com/](https://portal.authgear.com/). Or you can use your self-deployed Authgear.

From the Project listing, create a new Project or select an existing Project. After that, we will need to create an application in the project.

{% tabs %}
{% tab title="Portal" %}
**Step 1: Create an application in the Portal**

1. Go to **Applications** on the left menu bar.
2. Click **⊕Add Application** in the top tool bar.
3. Input the name of your application and select **Native App** as the application type. Click "Save".
4. You will see a list of guides that can help you for setting up, then click "Next".

![Create an application](../../.gitbook/assets/create-application-app-1.png)

**Step 2: Configure the application**

1. In your IDE (e.g. Visual Studio), define a custom URI scheme that the users will be redirected back to your app after they have authenticated with Authgear, e.g. `com.myapp.example://host/path`.\[^1]
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

## Create a Xamarin app

* Open Visual Studio
* Create a new project
* Choose the Xamarin.Forms template

## Install the SDK

* Search for "Authgear.Xamarin" on nuget.org and add it to your base project.\
  [https://www.nuget.org/packages/Authgear.Xamarin/](https://www.nuget.org/packages/Authgear.Xamarin/)
* Authgear.Xamarin targets MonoAndroid 12.0 on Android, and Xamarin.iOS10 on iOS. Update the target framework of the Android and iOS projects to match Authgear.Xamarin's target frameworks.
* Update Android and iOS project's Xamarin.Essentials to 1.7.2.

## Platform Integration

To finish the integration, setup the app to handle the redirectURI specified in the application. This part requires platform specific integration.

### Android

#### Define your own callback activity

```csharp
using System;

using Android.App;
using Android.Content.PM;
using Android.Runtime;
using Android.OS;
using Authgear.Xamarin;
using Xamarin.Forms;

namespace MyApp.Droid
{
    [Activity(NoHistory = true, LaunchMode = LaunchMode.SingleTop, Exported = true)]
    [IntentFilter(new[] { Android.Content.Intent.ActionView },
        Categories = new[] { Android.Content.Intent.CategoryDefault, Android.Content.Intent.CategoryBrowsable },
        DataScheme = "com.myapp.example")]
    public class WebAuthenticationCallbackActivity : Xamarin.Essentials.WebAuthenticatorCallbackActivity
    {
    }
}
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

#### Initialize a global AuthgearSdk instance

In your MainActivity.cs

```csharp
using System;

using Android.App;
using Android.Content.PM;
using Android.Runtime;
using Android.OS;

using Xamarin.Forms;
using Authgear.Xamarin;

namespace MyApp.Droid
{
    public class MainActivity : global::Xamarin.Forms.Platform.Android.FormsAppCompatActivity
    {
        protected override void OnCreate(Bundle savedInstanceState)
        {
            // ...

            var authgear = new AuthgearSdk(this, new AuthgearOptions
            {
                ClientId = CLIENT_ID,
                AuthgearEndpoint = ENDPOINT
            });
            DependencyService.RegisterSingleton<AuthgearSdk>(authgear);
            LoadApplication(new App());

            // ...
        }

        // other methods are omitted for brevity.
    }
}
```

### iOS

#### Add the following key-value pair in your iOS project Info.plist

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <!-- other entries ... -->
        <key>CFBundleURLTypes</key>
        <array>
          <dict>
            <key>CFBundleURLName</key>
            <string>com.myapp.example://host/path</string>

            <key>CFBundleURLSchemes</key>
            <array>
              <string>com.myapp.example</string>
            </array>

            <key>CFBundleTypeRole</key>
            <string>Editor</string>
          </dict>
        </array>
</dict>
</plist>
```

#### Initialize a global AuthgearSdk instance

In your AppDelegate.cs

```csharp
using Xamarin.Essentials;
using Xamarin.Forms;
using Authgear.Xamarin;

namespace MyApp.iOS
{
    [Register("AppDelegate")]
    public partial class AppDelegate : global::Xamarin.Forms.Platform.iOS.FormsApplicationDelegate
    {
        public override bool FinishedLaunching(UIApplication app, NSDictionary options)
        {
            global::Xamarin.Forms.Forms.Init();
            var authgear = new AuthgearSdk(app, new AuthgearOptions
            {
                ClientId = CLIENT_ID,
                AuthgearEndpoint = ENDPOINT
            });
            DependencyService.RegisterSingleton<AuthgearSdk>(authgear);
            LoadApplication(new App());

            return base.FinishedLaunching(app, options);
        }

        public override bool OpenUrl(UIApplication app, NSUrl url, NSDictionary options)
        {
            return Xamarin.Essentials.Platform.OpenUrl(app, url, options);
        }

        public override bool ContinueUserActivity(UIApplication application, NSUserActivity userActivity, UIApplicationRestorationHandler completionHandler)
        {
            if (Xamarin.Essentials.Platform.ContinueUserActivity(application, userActivity, completionHandler))
                return true;
            return base.ContinueUserActivity(application, userActivity, completionHandler);
        }
    }
}
```

## Try authenticate

### Edit your MainPage.xaml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="testauthgear.MainPage">

    <StackLayout>
        <Frame BackgroundColor="#2196F3" Padding="24" CornerRadius="0">
            <Label Text="Welcome to Xamarin.Forms!" HorizontalTextAlignment="Center" TextColor="White" FontSize="36"/>
        </Frame>
        <Button Text="Configure" Clicked="Configure_Clicked" />
        <Button Text="Authenticate" Clicked="Authenticate_Clicked"/>
    </StackLayout>

</ContentPage>
```

### Edit your MainPage.xaml.cs

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Xamarin.Forms;
using Authgear.Xamarin;

namespace MyApp
{
    public partial class MainPage : ContentPage
    {
        private AuthgearSdk authgear;


        public MainPage()
        {
            InitializeComponent();
            authgear = DependencyService.Get<AuthgearSdk>();
        }

        async void Configure_Clicked(object sender, EventArgs e)
        {
            // You must configure the instance before use.
            // Typically you should do this once on app launch.
            await authgear.ConfigureAsync();
        }

        async void Authenticate_Clicked(object sender, EventArgs e)
        {
            var userInfo = await authgear.AuthenticateAsync(new AuthenticateOptions
            {
                RedirectUri = REDIRECT_URI
            });
        }
    }
}
```

## Get the Logged In State

When you start launching the application. You may want to know if the user has logged in. (e.g. Show users the login page if they haven't logged in). The `SessionState` reflects the user logged in state in the SDK locally on the device. That means even the `SessionState` is `Authenticated`, the session may be invalid if it is revoked remotely. After initializing the Authgear SDK, call `FetchUserInfoAsync` to update the `SessionState` as soon as it is proper to do so.

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

The value of `SessionState` can be `Unknown`, `NoSession` or `Authenticated`. Initially, the `SessionState` is `Unknown`. After a call to `authgear.configure`, the session state would become `Authenticated` if a previous session was found, or `NoSession` if such session was not found.

## Fetching User Info

In some cases, you may need to obtain current user info through the SDK. (e.g. Display email address in the UI). Use the `FetchUserInfoAsync` function to obtain the user info, see [example](../../api-reference/apis/oauth-2.0-and-openid-connect-oidc/userinfo.md).

## Logout

To log out the user from the current app session, you need to invoke the`logout`function.

```csharp
await authgear.LogoutAsync();
```

## Calling An API

To include the access token to the HTTP requests to your application server, you set the bearer token manually by using `authgear.AccessToken`.

### Using HttpClient

You can get the access token through `authgear.AccessToken`. Call `RefreshAccessTokenIfNeededAsync` every time before using the access token, the function will check and make the network call only if the access token has expired. Then, include the access token into the Authorization header of the http request.

```csharp
await authgear.RefreshAccessTokenIfNeededAsync();
// Access token is ready to use
// AccessToken can be string or undefined
// It will be empty if user is not logged in or session is invalid
var accessToken = authgear.AccessToken;
var client = GetHttpClient();  // Get the re-used http client of your app, as per recommendation.
var httpRequestMessage = new HttpRequestMessage(myHttpMethod, myUrl);
httpRequestMessage.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
```

## Next steps

To protect your application server from unauthorized access. You will need to **integrate your backend with Authgear**.

{% content-ref url="../backend-api/" %}
[backend-api](../backend-api/)
{% endcontent-ref %}

## Xamarin SDK Reference

For detailed documentation on the Xamarin SDK, visit [Xamarin SDK Reference](https://authgear.github.io/authgear-sdk-xamarin/)

### Footnote

\[^1]: For further instruction on setting up custom URI scheme in Xamarin, see [https://www.xamarinhelp.com/uri-scheme/](https://www.xamarinhelp.com/uri-scheme/) \[^2]: For more explanation on JWT, see [https://en.wikipedia.org/wiki/JSON\_Web\_Token](https://en.wikipedia.org/wiki/JSON_Web_Token)
