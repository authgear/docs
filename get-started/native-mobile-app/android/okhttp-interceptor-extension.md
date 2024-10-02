# Android OKHttp Interceptor Extension (Optional)

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
