# Android OKHttp Interceptor Extension (Optional)

[![LLM | View as markdown](https://img.shields.io/badge/LLM-View%20as%20markdown-blue)](https://r.jina.ai/https://docs.authgear.com/get-started/native-mobile-app/android/okhttp-interceptor-extension)

The Authgear Android SDK provides an optional `Okhttp` interceptor which handles everything from refreshing the access token to putting the access token in the header.

## Get the Extension

The extension is included in the SDK. Please refer to the above section for getting the SDK.

## Usage

Configure `OkHttpClient` to use `AuthgearInterceptor` as follows:

```java
Authgear authgear = // Obtain the authgear instance.
OKHttpClient client = new OkHttpClient.Builder()
            .addInterceptor(AuthgearInterceptor(authgear))
            .build()
```

The client would then include the access token in every request and refresh the access token when necessary before the requests.
