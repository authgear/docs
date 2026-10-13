---
description: >-
  Passkeys give users a simple and secure way to sign in to your apps and
  websites across platforms without passwords.
---

# Add Passkeys Login

Passkeys replace passwords and other passwordless login methods. They are built on WebAuthn, part of the FIDO2 standard, which uses public key cryptography to authenticate the user.

#### Platform support and multiple devices

Passkeys are supported by current versions of the major browsers and operating systems, including iOS 16 and macOS 13 (Ventura) or later, and Android 9 (API level 28) or later. See [passkeys.dev](https://passkeys.dev/device-support/) for the full list.

Passkeys sync through the platform's password manager, such as iCloud Keychain or Google Password Manager, so users can sign in on all their devices. Users confirm a sign-in with the device's screen lock, such as Face ID, Touch ID, Windows Hello, or an Android fingerprint or face unlock.

#### Hardware security keys

Passkeys can also be stored in hardware security keys such as [YubiKeys](https://www.yubico.com/blog/passkeys-and-the-future-of-modern-authentication/), which provide the highest security against attacks.

## Add Passkeys to your apps with Authgear

To enable passkeys:

1. In your project portal, go to **Authentication > Login Methods**.
2. In the **Passkey** section, turn on **Enable passkey support for compatible devices.**
3. Press **Save**.

In the same section, **Show "Don't ask me again" option on the passkey upsell screen** lets users opt out of future reminders to create a passkey. A user who opts out can still create one in their account settings.

{% hint style="info" %}
Keep another login method enabled, such as **Password** or **Passwordless**, for users whose devices or browsers do not support passkeys.

To use only passkeys, select **Custom**, and in the **Custom Login Methods** tab keep at least one Login ID active and deactivate every authenticator. Keep **Enable passkey support for compatible devices.** turned on.
{% endhint %}

## Use passkeys in your iOS or Android app

When your mobile app shows the Authgear UI in its own WebView instead of the system browser (see [Using WebView to open the Authgear UI](../../integration/mobile-apps/using-webview-to-open-the-authgear-ui.md)), the operating system allows passkeys only for apps that your project lists. Authgear publishes the listed apps in `/.well-known/apple-app-site-association` and `/.well-known/assetlinks.json` on your project's domain.

Apps opening the Authgear UI in the system browser need none of this.

{% hint style="warning" %}
List only apps you control. A listed app controls everything shown in its WebView, including the Authgear UI, and can request your users' passkeys.
{% endhint %}

### List your apps

1. In your project portal, go to **Applications** and open your Native App application.
2. In the **Associated apps** section, press **Add iOS app** or **Add Android app**.
3. Enter the app's details:
   * **iOS:** the Team ID from your Apple Developer account, and the Bundle ID.
   * **Android:** the package name, and the SHA-256 fingerprint of every certificate the app is distributed with, one per line.
4. Press **Save**.

An app can be listed only once, across all applications in the project. A listed app can use passkeys whichever application it signs in with.

### iOS

Add the entry shown under **iOS Associated Domains entry**, `webcredentials:<your project domain>`, to the Associated Domains capability of your app in Xcode.

Apple fetches `apple-app-site-association` through its own CDN, so a change can take time to reach devices. To test a change sooner, append `?mode=developer` to the entry, for example `webcredentials:auth.example.com?mode=developer`. Apple then fetches the file directly, but only for a development-signed build on a device with Developer Mode turned on.

Requires iOS 16 or later.

### Android

* **Include the Play App Signing certificate.** An app installed from Google Play is signed with it, not with your upload certificate. Find its fingerprint on the **App signing** page in Google Play Console. A build signed with a certificate you have not listed cannot use passkeys.
* **List a debug certificate only in a project used for development.** A debug keystore has a well-known password, so anyone who obtains it can sign an app that passes as yours.
* **Do not declare a verified intent filter for your project's domain.** The association file also lets a listed app open links on that domain, including the Authgear UI and links in emails, if the app declares a verified intent filter for it. Check the app's merged manifest, because a library can add one.

Requires an Android System WebView that supports WebAuthn, and Google Play services 24.07 or later. On other devices, users sign in with your other login methods.
