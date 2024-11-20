---
description: >-
  Guide on how to fix SubtleCrypto: digest() non-HTTPS/Secure context Error in
  Authgear SDK
---

# How to Fix SubtleCrypto: digest() undefined Error in Authgear SDK

The Authgear SDK uses a browser API [SubtleCrypto: digest()](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/digest) which requires HTTPS. As a result, if your client application that uses the SDK is on HTTP (non-secure), you may run into the SubtleCrypto: digest() non-HTTPS/secure context error.

To use the SDK, your client application must be on HTTPS or be running in **secure contexts**. Secure contexts include running your application from `localhost` or `127.0.0.1`.

The following are examples of the error messages you will get SubtleCrypto: digest() non-HTTPS/secure context error in Chrome and Firefox.

**Chrome**

```json
{
  "stack": "TypeError: Cannot read properties of undefined (reading 'digest')\n    at http://authgeartest.online/index.9bf4411c.js:1:219150\n    at p (http://authgeartest.online/index.9bf4411c.js:1:148005)\n    at o._invoke (http://authgeartest.online/index.9bf4411c.js:1:147765)\n    at Generator.next (http://authgeartest.online/index.9bf4411c.js:1:148364)\n    at e_ (http://authgeartest.online/index.9bf4411c.js:1:145029)\n    at i (http://authgeartest.online/index.9bf4411c.js:1:145227)\n    at http://authgeartest.online/index.9bf4411c.js:1:145288\n    at new Promise (<anonymous>)\n    at http://authgeartest.online/index.9bf4411c.js:1:145168\n    at iz (http://authgeartest.online/index.9bf4411c.js:1:219441)",
  "message": "Cannot read properties of undefined (reading 'digest')"
}
```

**Firefox**

```json
{
  "fileName": "http://you-app-url.com/index.9bf4411c.js",
  "lineNumber": 1,
  "columnNumber": 219125,
  "message": "window.crypto.subtle is undefined"
}
```

### Fix

The following are ways to fix the SubtleCrypto: digest() non-HTTPS/secure context error:

* If you need to run your application on a local environment or for testing where you can't set up HTTPS, consider running your application in [secure contexts](https://developer.mozilla.org/en-US/docs/Web/Security/Secure\_Contexts#when\_is\_a\_context\_considered\_secure), For example, http://localhost or http://127.0.0.1.
* To run your application from another domain other than a localhost, set up HTTPS on the domain.

You can learn more about `SubtleCrypto: digest()` and Secure Contexts see the pages linked below:

* [https://developer.mozilla.org/en-US/docs/Web/API/Crypto/subtle](https://developer.mozilla.org/en-US/docs/Web/API/Crypto/subtle)
* [https://developer.mozilla.org/en-US/docs/Web/Security/Secure\_Contexts](https://developer.mozilla.org/en-US/docs/Web/Security/Secure\_Contexts)
