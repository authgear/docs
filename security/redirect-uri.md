---
description: Implication of using non-HTTP scheme in redirect URI.
---

# Non-HTTP scheme redirect URI

If your application is a mobile application, you can choose to use custom URI scheme in the redirect URI. Custom URI scheme redirect URI does not require verification. Therefore, it is possible that there is a malicious application installed on the user's device, which claims itself capable of handling your redirect URI.

The whole attack is as follows:

1. The attacker generates a [code verifier](https://tools.ietf.org/html/rfc7636#section-4.1) and a [code challenge](https://tools.ietf.org/html/rfc7636#section-4.2).
2. The attacker somehow makes the victim to install a malicious application on their device. The malicious application registers the same redirect URI that your application is using.
3. The victim somehow is mislead by the attacker to visit the authorization endpoint with the code challenge generated in Step 1.
4. The victim completes the authorization code flow and the malicious application receives the authorization code.
5. The malicious application uses the code verifier in Step 1 and the just received authorization code to exchange for an refresh token and an access token.

It is impossible to prevent malicious applications being installed on the user's device. However, we can mitigate this attack by not using custom URI scheme.

To do so, you can use [Apple Universal Links](https://developer.apple.com/documentation/xcode/allowing\_apps\_and\_websites\_to\_link\_to\_your\_content) and [Android App Links](https://developer.android.com/training/app-links).
