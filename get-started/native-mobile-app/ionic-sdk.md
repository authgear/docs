---
description: Guide on how to use Authgear in an Ionic project
---

# Ionic SDK

In this post, you'll learn how to use Authgear with your Ionic project using the Authgear Ionic SDK.

### Objectives (What we'll build)

At the end of this tutorial, we'll build an Ionic app that can do the following:

* Allow users to log in to their account on your Authgear project
* Allow new users to sign up
* Allow signed-in users to view their user info and logout.

The final UI for the app we'll build should look like this:

<figure><img src="../../.gitbook/assets/authgear-ionic-example-app.png" alt=""><figcaption><p>authgear ionic example app landing page</p></figcaption></figure>

### Pre-requisites

To follow this guide seamlessly, make sure to have the following:

* Node.js installed on your local machine
* Android Studio (for building the Android client of your application)
* X-code (for building the iOS client of your application)
* An Authgear account. You can sign up for one for free [here](https://authgear.com/).
* Any code editor (VS Code, Sublime, etc)

Now let us get into the steps for using Authgear in an Ionic project.

### Part 1: Configure your Authgear Application

In this part, you'll learn how to configure an Authgear application so that you can use it in your Ionic project. You'll do this by performing the following steps in the Authgear Portal.

#### Step 1: Set up an Authgear Application

First, log in to Authgear Portal at [https://portal.authgear.com/](https://portal.authgear.com/) and select an existing project or create a new one.

In your project, navigate to the **Applications** section then click on **Add Application** to create a new Authgear application. Enter a name for your application and select **Native App** as the Application Type. Next, click **Save** to continue to the configuration page for your new application.

<figure><img src="../../.gitbook/assets/authgear-new-app-native copy.png" alt=""><figcaption></figcaption></figure>

#### Step 2: Add Authorized Redirect URIs

In this step, you'll set up authorized redirect URIs for your application. An authorized redirect URI should be a URI pointing to a page on your Ionic app where you want to redirect users at the end of the authorization flow.

To add a URI, scroll to the **URIs** section of your application configuration page and enter the URI in the text field. You can click the **Add URI** button to add additional URIs.

For our example app, add the following URIs:

* `com.authgear.example.capacitor://host/path`
* `capacitor://localhost`
* `http://localhost:8100/oauth-redirect`
* `https://localhost`

<figure><img src="../../.gitbook/assets/authgear-config-redirect-uri.png" alt=""><figcaption><p>authgear-app-redirect-uris</p></figcaption></figure>

Once you're done, click on the **Save** button.

### Part 2: Implement Ionic App

Now that we have our Authgear application configured, we can now proceed with creating the Ionic application that will have all the features stated in our objective earlier.

For this tutorial, we'll be implementing an Ionic app using React.

#### Step 1: Create Ionic Project

Before you can create an Ionic project, install the Ionic CLI on your computer by running the following command in Terminal or Command Prompt:

```sh
npm install -g @ionic/cli native-run cordova-res
```

Now create a new Ionic project by running the following command:

```sh
ionic start authgear-ionic-example --type=react --capacitor
```

After running the above command, follow the wizard to create a new **blank project**.

Next, open your new project in a code editor and update for `appId` in **capacitor.config.ts** to the following value:

```typescript
appId: 'com.authgear.example.capacitor',
```

This new value for `appId` is the same value we used in the authorized redirect URI earlier.

**Note:** It is important that you update the value for `appId` before you create the Android and iOS projects for your Ionic application. Doing this will enable Capacitor to create your Android and iOS project with the value for appId as the package name and app ID.

You can run the `ionic serve` command to preview your new blank project on a browser.

Finally, create the Android and iOS projects for your app by running the following commands from your Ionic project's root folder:

First, install the Android and iOS platforms:

```sh
npm i @capacitor/android @capacitor/ios
```

Then, create the projects:

```sh
npx cap add android
npx cap add ios
```

#### Step 2: Install Authgear SDK

In this step, you'll install the Authgear SDK for Ionic (Capacitor) and the Javascript SDK for the web. The web SDK will help you test your application on a web browser.

To install the SDKs, run the following commands in your Terminal or Command Prompt:

**Authgear Ionic SDK**

```bash
npm i @authgear/capacitor
```

**Authgear Web SDK**

```sh
npm i @authgear/web
```

#### Step 3: Configure Authgear SDK

In this step, you'll learn how to configure your Ionic project using the details from your Authgear application configuration.

To get started, open **src/pages/Home.tsx** in your code editor and import the Authgear SDK by adding the following code to the top of the file:

```typescript
import authgearWeb, {
  SessionState,
  CancelError as WebCancelError,
  UserInfo,
} from "@authgear/web";
import authgearCapacitor, {
  CancelError as CapacitorCancelError,
} from "@authgear/capacitor";
```

The above code imports all the components of the Authgear SDK we need for our example app.

Because Ionic apps can run on the web and native mobile platforms, we have to import both Authgear web and Authgear Capacitor SDKs.

Next, add the following constants to **Home.tsx** just below the last import statement:

```typescript
const CLIENT_ID = "";
const ENDPOINT = "";
const REDIRECT_URI_WEB_AUTHENTICATE = "http://localhost:8100/oauth-redirect";
const REDIRECT_URI_CAPACITOR = "com.authgear.example.capacitor://host/path";
const REDIRECT_URI_WEB_REAUTH = "http://localhost:8100/reauth-redirect";
```

Update the values for the constants (`CLIENT_ID`, `ENDPOINT`) you just added to the correct values from your application's configuration page in the Authgear Portal.

Also, add this small utility function that will help to check whether your Ionic app is running natively or on a web browser:

```typescript
function isPlatformWeb(): boolean {
  return Capacitor.getPlatform() === "web";
}
```

Now implement a new `AuthenticationScreen` component in **Home.tsx** by pasting the following code:

```typescript
function AuthenticationScreen() {

  const [isAlertOpen, setIsAlertOpen] = useState(false);
  const [alertHeader, setAlertHeader] = useState("");
  const [alertMessage, setAlertMessage] = useState("");

  const [loading, setLoading] = useState(false);
  const [initialized, setInitialized] = useState(false);

  const showError = useCallback((e: any) => {
    const json = JSON.parse(JSON.stringify(e));
    json["constructor.name"] = e?.constructor?.name;
    json["message"] = e?.message;
    let message = JSON.stringify(json);

    if (e instanceof WebCancelError || e instanceof CapacitorCancelError) {
      // Cancel is not an error actually.
      return;
    }

    setIsAlertOpen(true);
    setAlertHeader("Error");
    setAlertMessage(message);
  }, []);

  const postConfigure = useCallback(async () => {
    const sessionState = isPlatformWeb()
      ? authgearWeb.sessionState
      : authgearCapacitor.sessionState;
    if (sessionState !== "AUTHENTICATED") {
      setInitialized(true);
      return;
    }

    if (isPlatformWeb()) {
      await authgearWeb.fetchUserInfo();
    } else {
      await authgearCapacitor.fetchUserInfo();
    }

    setInitialized(true);
  }, []);

  const configure = useCallback(async () => {
    setLoading(true);
    try {

      if (isPlatformWeb()) {
        await authgearWeb.configure({
          clientID: CLIENT_ID,
          endpoint: ENDPOINT,
          sessionType: "refresh_token",
          isSSOEnabled: false,
        });
      } else {
        await authgearCapacitor.configure({
          clientID: CLIENT_ID,
          endpoint: ENDPOINT
        });
      }
      await postConfigure();
    } catch (e) {
      showError(e);
    } finally {
      setLoading(false);
    }
  }, [
    CLIENT_ID,
    ENDPOINT
  ]);
}
```

#### Step 4: Start Authentication

Here, we'll implement an `authenticate()` method inside the `AuthenticationScreen` component we created in the previous step.

To do this, first, add the following constants that the method will depend on to the top of `AuthenticationScreen` component:

```typescript
const [page, setPage] = useState("");

const [sessionState, setSessionState] = useState<SessionState | null>(() => {
  if (isPlatformWeb()) {
    return authgearWeb.sessionState;
  }
  return authgearCapacitor.sessionState;
});

const loggedIn = sessionState === "AUTHENTICATED";
```

Then, add the `authenticate()` method by pasting the following code to the end of the `AuthenticationScreen` component:

```typescript
const authenticate = useCallback(async (page: string) => {
    setLoading(true);
    try {
      if (isPlatformWeb()) {
        authgearWeb.startAuthentication({
          redirectURI: REDIRECT_URI_WEB_AUTHENTICATE,
          page: page,
        });
      } else {
        const result = await authgearCapacitor.authenticate({
          redirectURI: REDIRECT_URI_CAPACITOR,
          page: page,
        });
        showUserInfo(result.userInfo);
      }
    } catch (e) {
      showError(e);
    } finally {
      setLoading(false);
    }
  }, [showUserInfo, page]);
```

Calling this `authenticate()` method will initialize an authentication flow. The `page` parameter can be used to specify whether to start the authentication flow on the login page or signup page.

#### Step 5: Handle Redirect in App

At the end of an authentication flow, your users will be redirected to the URL you specified in `redirectURI`. In this step, we'll set up the routes and code to process redirects to the URIs.

First, create a new file **OAuthRedirect.tsx** in **src/pages/** and add the following code to it:

```typescript
import { useCallback, useEffect } from "react";
import authgearWeb from "@authgear/web";
import { useIonRouter } from "@ionic/react";

export default function OAuthRedirect() {
  const router = useIonRouter();

  const finishAuthentication = useCallback(async () => {
    const CLIENT_ID = "";
    const ENDPOINT = "";

    try {
      await authgearWeb.configure({
        clientID: CLIENT_ID,
        endpoint: ENDPOINT,
        sessionType: "refresh_token",
      });
      await authgearWeb.finishAuthentication();
      router.push("/", "root", "replace");
    } catch (e) {
      console.error(e);
    }
  }, [router]);

  useEffect(() => {
    finishAuthentication();
  }, [finishAuthentication]);

  return (
    <div>
      Finishing authentication. Open the inspector to see if there is any error.
    </div>
  );
}
```

Change the values for `CLIENT_ID` and `ENDPOINT` in the above code to the correct value from your Authgear application configuration page.

Now open **src/App.tsx** and create a new route for `OAuthRedirect` using the following code:

```typescriptreact
<Route exact path="/oauth-redirect">
    <OAuthRedirect />
</Route>
```

Remember to import `OAuthRedirect` in **App.tsx**.

To handle redirect in the Android project, add the following code to **android/app/src/main/AndroidManifest.xml**:

```xml
<!-- Authgear SDK -->
<activity
    android:name="com.authgear.capacitor.OAuthRedirectActivity"
    android:launchMode="singleTask"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <!-- Configure data to be the exact redirect URI your app uses. -->
        <!-- NOTE: The redirectURI supplied in AuthenticateOptions has to match as well -->
        <data
            android:host="host"
            android:pathPrefix="/path"
            android:scheme="com.authgear.example.capacitor" />
    </intent-filter>
</activity>
```

#### Step 6: Implement UI

At this point, you will create the User Interface for the AuthenticationScreen component.

To do that, add the following code to the end of the `AuthenticationScreen` component method:

```typescript
const fetchUserInfo = useCallback(async () => {
    setLoading(true);
    try {
      if (isPlatformWeb()) {
        const userInfo = await authgearWeb.fetchUserInfo();
        showUserInfo(userInfo);
      } else {
        const userInfo = await authgearCapacitor.fetchUserInfo();
        showUserInfo(userInfo);
      }
    } catch (e) {
      showError(e);
    } finally {
      setLoading(false);
    }
  }, [showError, showUserInfo]);

  const logout = useCallback(async () => {
    setLoading(true);
    try {
      if (isPlatformWeb()) {
        await authgearWeb.logout({
          redirectURI: window.location.origin + "/",
        });
      } else {
        await authgearCapacitor.logout();
      }
    } catch (e) {
      showError(e);
    } finally {
      setLoading(false);
    }
  }, [showError]);

  // On web, it is more natural to configure automatically if client ID and endpoint are filled in.
  useEffect(() => {
      configure();
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  const onAlertDismiss = useCallback(
    (_e: IonAlertCustomEvent<OverlayEventDetail>) => {
      setIsAlertOpen(false);
    },
    []
  );

  const onClickAuthenticate = useCallback(
    (e: MouseEvent<HTMLIonButtonElement>, page: string) => {
      e.preventDefault();
      e.stopPropagation();

      authenticate(page);
    },
    [authenticate]
  );

  const onClickFetchUserInfo = useCallback(
    (e: MouseEvent<HTMLIonButtonElement>) => {
      e.preventDefault();
      e.stopPropagation();

      fetchUserInfo();
    },
    [fetchUserInfo]
  );

  const onClickLogout = useCallback(
    (e: MouseEvent<HTMLIonButtonElement>) => {
      e.preventDefault();
      e.stopPropagation();

      logout();
    },
    [logout]
  );

  return (
    <>
    <IonAlert
        isOpen={isAlertOpen}
        header={alertHeader}
        message={alertMessage}
        onIonAlertDidDismiss={onAlertDismiss}
      />
    <div className="container">
        <h1>
          Welcome
        </h1>
        <p>Tap any of the buttons to below to login or signup</p>
        <IonButton
          className="button"
          disabled={!initialized || loading || loggedIn}
          onClick={
            (event) => {
              setPage("signup");
              onClickAuthenticate(event, "login")
            }
          }
        >
          Login
        </IonButton>
        <IonButton
          className="button"
          disabled={!initialized || loading || loggedIn}
          onClick={(event) => {
            onClickAuthenticate(event, "signup")
          }}
        >
          Signup
        </IonButton>

        <IonButton
          className="button"
          disabled={!initialized || loading || !loggedIn}
          onClick={onClickFetchUserInfo}
        >
          Fetch User Info
        </IonButton>

        <IonButton
          className="button"
          disabled={!initialized || loading || !loggedIn}
          onClick={onClickLogout}
        >
          Logout
        </IonButton>
        
      </div>
    </>
  );
```

#### Step 7: Deploy app to mobile

To deploy your app to a mobile device (for example Android) run the following commands:

First build your project by running:

```sh
npm run build
```

Then sync the changes to the mobile project using this command:

```sh
npx cap sync
```

You can run the project by opening the `android` project folder in Android Studio or `ios` folder in X-code.

You can quickly open the project in Android Studio using the following command:

```sh
npx cap open android
```

Once your project builds successfully, you can try the Login, Signup, Fetch User Info, and Logout buttons.

### Conclusion

Authgear Capacitor SDK makes it easier to use Authgear in your Ionic application. It provides many helpful methods and interfaces for interacting with the Authgear service from your Ionic application. To learn more about the SDK check [the SDK Reference](https://authgear.github.io/authgear-sdk-js/docs/capacitor/). Also, check out the complete repo for the Authgear Ionic SDK example app [here](https://github.com/authgear/authgear-sdk-js/tree/master/example/capacitor).
