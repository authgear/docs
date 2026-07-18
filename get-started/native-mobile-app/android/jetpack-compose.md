---
description: Add Authgear to an Android app built with Jetpack Compose
---

# Android SDK with Jetpack Compose

This is a minimal **Jetpack Compose** version of the [Android SDK guide](README.md). It produces the same login → user info → settings → logout flow using Compose and the SDK's Kotlin coroutine (`suspend`) API instead of XML layouts and callback listeners.

{% hint style="info" %}
Set up your Authgear application and Redirect URI first by following [Setup Application in Authgear](README.md#setup-application-in-authgear) in the main guide. This page reuses the same **Client ID**, **Endpoint**, and Redirect URI (`com.example.authgeardemo://host/path`).

A fuller, production-shaped version of this app (with a `ViewModel` + `StateFlow`) is available in the [example repo](https://github.com/authgear/authgear-example-android). This page keeps things minimal so the Authgear integration stays front and center.
{% endhint %}

## Step 1: Create a Compose project

Open Android Studio and create a new project with **Empty Activity** — the current template uses Jetpack Compose and sets up the Compose dependencies for you. Feel free to skip this step if you are adding Authgear to an existing Compose app.

## Step 2: Add the Authgear SDK

Follow [Step 2 of the main guide](README.md#step-2-add-authgear-sdk-to-your-project) to add the Maven Central dependency and enable core library desugaring. In short, in your app-level `build.gradle`:

```groovy
android {
    compileOptions {
        coreLibraryDesugaringEnabled true
    }
}

dependencies {
    // Other implementations
    implementation 'com.authgear:android-sdk:3.0.0'
    coreLibraryDesugaring 'com.android.tools:desugar_jdk_libs:2.0.3'
}
```

## Step 3: Use an AppCompat app theme

The SDK's authentication screens (`OAuthActivity`, `OAuthRedirectActivity`) are `AppCompatActivity`. Your app's **window theme** must descend from `Theme.AppCompat`, otherwise login crashes with *"You need to use a Theme.AppCompat theme (or descendant) with this activity"*. Your Compose UI still themes itself — this is only the window theme.

Open `res/values/themes.xml` and make sure your app theme uses an AppCompat (or Material 3) parent:

```xml
<resources>
    <style name="Theme.MyDemoApp" parent="Theme.AppCompat.DayNight.NoActionBar" />
</resources>
```

Confirm the `<application android:theme="@style/Theme.MyDemoApp">` entry in `AndroidManifest.xml` points to it.

## Step 4: Initialize Authgear

Create the Authgear instance in your `MainActivity` and host your Compose UI:

```kotlin
class MainActivity : ComponentActivity() {

    private lateinit var authgear: Authgear

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        authgear = Authgear(application, "<CLIENT_ID>", "<AUTHGEAR_ENDPOINT>")
        setContent {
            MaterialTheme {
                Surface(modifier = Modifier.fillMaxSize()) {
                    MainScreen(authgear)
                }
            }
        }
    }
}
```

Replace `<CLIENT_ID>` and `<AUTHGEAR_ENDPOINT>` with the values from the configuration page of your Authgear client application.

## Step 5: Build the screen

The SDK exposes `suspend` functions (`configure`, `authenticate`, `fetchUserInfo`, `logout`) that you call from Compose with `rememberCoroutineScope()` and `LaunchedEffect`. This single composable configures Authgear when it first appears (restoring an existing session), then shows a **Login** button or the logged-in view based on `sessionState`:

```kotlin
private const val REDIRECT_URI = "com.example.authgeardemo://host/path"
private const val TAG = "AuthgearDemo"

@Composable
fun MainScreen(authgear: Authgear) {
    val scope = rememberCoroutineScope()
    var configuring by remember { mutableStateOf(true) }
    var email by remember { mutableStateOf<String?>(null) }
    var busy by remember { mutableStateOf(false) }

    // Configure once when the screen first appears; restore an existing session.
    LaunchedEffect(Unit) {
        try {
            authgear.configure()
            if (authgear.sessionState == SessionState.AUTHENTICATED) {
                email = authgear.fetchUserInfo().email
            }
        } catch (e: Throwable) {
            Log.e(TAG, "configure failed", e)
        }
        configuring = false
    }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(24.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center,
    ) {
        when {
            configuring -> CircularProgressIndicator()

            email != null -> {
                Text("Welcome, $email")
                OutlinedButton(
                    enabled = !busy,
                    onClick = { authgear.open(Page.SETTINGS) },
                    modifier = Modifier.padding(top = 16.dp),
                ) { Text("User Settings") }
                Button(
                    enabled = !busy,
                    onClick = {
                        scope.launch {
                            busy = true
                            try {
                                authgear.logout()
                                email = null
                            } catch (e: Throwable) {
                                Log.e(TAG, "logout failed", e)
                            }
                            busy = false
                        }
                    },
                ) { Text("Logout") }
            }

            else -> {
                Button(
                    enabled = !busy,
                    onClick = {
                        scope.launch {
                            busy = true
                            try {
                                email = authgear.authenticate(AuthenticateOptions(REDIRECT_URI)).email
                            } catch (e: Throwable) {
                                Log.e(TAG, "login failed", e)
                            }
                            busy = false
                        }
                    },
                ) { Text("Login") }
            }
        }
    }
}
```

{% hint style="info" %}
`configure`, `authenticate`, `logout`, and `fetchUserInfo` are `suspend` extension functions on `Authgear` — import them (e.g. `import com.oursky.authgear.configure`) if they show as unresolved. Import any other class that shows as unresolved as well.
{% endhint %}

## Step 6: Handle the Redirect URI

Add the `OAuthRedirectActivity` to your `AndroidManifest.xml` exactly as in [Step 6 of the main guide](README.md#step-6-setup-redirect-uri-for-your-android-app), matching the scheme/host/path of your Redirect URI. If your app targets Android 11+ (API level 30+), also add the `<queries>` block from that section.

Run the app, tap **Login**, and complete the flow in the browser. On success you'll see the welcome text with the user's email, plus the **User Settings** and **Logout** buttons.

## Next steps

* Get the access token and call your backend — see [Using the Access Token in HTTP Requests](README.md#using-the-access-token-in-http-requests).
* For a version structured with a `ViewModel`, `StateFlow`, and a testable seam over the SDK, see the [example app](https://github.com/authgear/authgear-example-android).
