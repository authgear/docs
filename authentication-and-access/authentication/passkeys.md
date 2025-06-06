---
description: >-
  Passkeys give users a simple and secure way to sign in to your apps and
  websites across platforms without passwords.
---

# Add Passkeys Login

Passkeys replace passwords and other passwordless login methods. It is built on the WebAuthn standard (also known as FIDO Sign-in), which uses public key cryptography to authenticate the user. With 1 click, Authgear upgrades your app to support this cutting-edge auth technology.

#### Platform support and Multi-devices

The passkey standard is supported on the latest versions of Chrome, Safari, and Firefox browsers. On iOS 16 and macOS 13 (Ventura), [Apple has added passkey support](https://developer.apple.com/passkeys/) to the iCloud Keychain service. Passkeys are also [supported on Android 9](https://developers.google.com/identity/passkeys/supported-environments#android-passkey-support) (API level 28) or higher. A passkey is synchronized and relayed with an iCloud account and can be used across a user's devices.&#x20;

Users can log in to their accounts using their biometrics easily. On Apple devices, Touch ID and Face ID authorize the use of the passkey which then authenticates the user on the app or website.

#### Hardware security keys

Besides the built-in support of all major desktop and mobile platforms, passkeys can also be stored in hardware security keys such as [YubiKeys](https://www.yubico.com/blog/passkeys-and-the-future-of-modern-authentication/), which provide the highest security against attacks.

## Add Passkeys to your apps with Authgear

Authgear adds a passkey feature to your apps and websites instantly. To enable it:

1. In your project portal, go to **Authentication > Login Methods**.
2. In the **Select Login Methods** section, turn on the **Enable passkey support for compatible devices.** toggle.
3. Press "Save" and your app now supports passkey login!

{% hint style="info" %}
It will take time for the passkey technology to be available on everyone's devices. In the transition stage, it is recommended to enable "Password" or "Passwordless via Email/Phone" in your project so users with non-compatible devices can access your app.

If you want to use ONLY passkeys in your app, it's perfectly supported too! Select **Custom** and deactivate all authenticators in the **Custom Login Methods** tab. Remember to keep **Enable Passkey support for compatible devices.** turned on.
{% endhint %}

## Support on different platforms

See the list of Passkey support via Authgear on different platforms.

* macOS 12: Passkey is supported on major browsers. However, the credentials are deleted when clearing browser data.
* iOS 15.5: Passkey is supported on Safari and stored locally a the device. Credential will be deleted by "Settings > Safari > Clear History and Website Data"
* iOS 16 Beta 3: Passkey is synced with iCloud Keychain. The individual credentials can be viewed and managed in "Settings > Passwords"
* Android 9 (API level 28) or higher: Supported.
