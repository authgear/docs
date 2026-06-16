---
description: Protect a NestJS API with Authgear using the @authgear/nestjs SDK
---

# NestJS

Protect a [NestJS](https://nestjs.com/) API with Authgear using the `@authgear/nestjs` SDK. The SDK validates Authgear **JWT access tokens** offline (via OIDC discovery and JWKS) and gives you a NestJS module, an authentication guard, and decorators — so protecting a route takes just a few lines.

A complete example application is available at [authgear/authgear-example-nestjs](https://github.com/authgear/authgear-example-nestjs).

{% hint style="info" %}
This SDK is for the **resource server** side — it verifies the access tokens your API receives. It does not perform the login flow. Your users log in through a separate client (a [Single-Page App](../single-page-app/), a [mobile app](../native-mobile-app/), or any OAuth client), which then calls your NestJS API with the access token.
{% endhint %}

**What you will build:**

* A public route (`GET /health`) that needs no token
* A protected route (`GET /me`) that returns the authenticated user's claims

***

### Setting Up Your Application in Authgear

#### Step 1: Enable JWT access tokens

The SDK validates JWT access tokens offline, so the application that issues the tokens your API will accept must issue JWTs.

1. Sign in to the [Authgear Portal](https://portal.authgear.com/)
2. Select your Project, then go to **Applications** and open the application your client uses to log in (or [create one](../single-page-app/) — e.g. a **Single Page Application**)
3. Under the **Access Token** section, turn on **Issue JWT as access token**
4. Note down your **Endpoint** (e.g. `https://your-project.authgear.cloud`) and **Client ID** — you will need these shortly
5. Click **Save**

{% hint style="warning" %}
Without **Issue JWT as access token**, Authgear issues opaque access tokens, which this SDK cannot validate offline. Requests would be rejected with `401`.
{% endhint %}

***

### Building Your NestJS Application

#### Step 1: Create a NestJS Project

```bash
npm i -g @nestjs/cli
nest new my-api
cd my-api
```

#### Step 2: Install the Authgear SDK

```bash
npm install @authgear/nestjs
```

This also installs `@nestjs/config`, which we use to read configuration from the environment:

```bash
npm install @nestjs/config
```

#### Step 3: Configure Environment Variables

Create a `.env` file in the project root:

```bash
AUTHGEAR_ENDPOINT=https://your-project.authgear.cloud
AUTHGEAR_CLIENT_ID=your-client-id
```

{% hint style="info" %}
`AUTHGEAR_CLIENT_ID` is optional. When set, the SDK additionally asserts that the token's `client_id` claim matches it.
{% endhint %}

#### Step 4: Register the Authgear Module

Register `AuthgearModule` in your root module. Setting `global: true` registers the guard as a global guard, so **every route is protected by default** unless explicitly marked public.

```typescript
// src/app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { AuthgearModule } from '@authgear/nestjs';
import { AppController } from './app.controller';

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    AuthgearModule.forRootAsync({
      global: true,
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        endpoint: config.getOrThrow<string>('AUTHGEAR_ENDPOINT'),
        clientID: config.get<string>('AUTHGEAR_CLIENT_ID'),
      }),
    }),
  ],
  controllers: [AppController],
})
export class AppModule {}
```

{% hint style="info" %}
If your configuration is static, you can use `AuthgearModule.forRoot({ endpoint: '...', global: true })` instead of `forRootAsync`.
{% endhint %}

#### Step 5: Protect Your Routes

With the guard registered globally, mark public routes with `@Public()`. Read the authenticated user with the `@CurrentUser()` parameter decorator.

```typescript
// src/app.controller.ts
import { Controller, Get } from '@nestjs/common';
import { Public, CurrentUser, AuthgearClaims } from '@authgear/nestjs';

@Controller()
export class AppController {
  // Public — no token required
  @Public()
  @Get('health')
  health() {
    return { status: 'ok' };
  }

  // Protected — requires a valid Authgear access token
  @Get('me')
  me(@CurrentUser() user: AuthgearClaims) {
    return {
      sub: user.sub,
      isVerified: user.isVerified,
      isAnonymous: user.isAnonymous,
    };
  }
}
```

`AuthgearClaims` exposes the common claims (`sub`, `iss`, `aud`, `clientID`, `isVerified`, `isAnonymous`, `canReauthenticate`) plus the full decoded payload as `raw` for any custom claims.

{% hint style="info" %}
Prefer to protect routes individually instead of globally? Omit `global: true` and apply the guard per controller or handler with `@UseGuards(AuthgearAuthGuard)`.
{% endhint %}

***

### Running the Application

```bash
npm run start:dev
```

The API listens on [http://localhost:3000](http://localhost:3000).

***

### Testing the Integration

The public route works without a token:

```bash
curl -i http://localhost:3000/health
# 200 {"status":"ok"}
```

The protected route is rejected without a valid token:

```bash
curl -i http://localhost:3000/me
# 401 {"message":"Missing bearer token","error":"Unauthorized","statusCode":401}
```

To call the protected route, send an Authgear access token as a Bearer token:

```bash
curl -i http://localhost:3000/me \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
# 200 {"sub":"...","isVerified":true,"isAnonymous":false}
```

{% hint style="info" %}
Obtain an access token by logging a user in through a client application. The [example project](https://github.com/authgear/authgear-example-nestjs) includes a small frontend that signs in and calls the protected API for you. To build your own client, see the [Single-Page App](../single-page-app/) or [Native/Mobile App](../native-mobile-app/) guides.
{% endhint %}

***

### Module Options

`forRoot()` and the object returned by the `forRootAsync()` factory accept:

| Option | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `endpoint` | `string` | ✓ | — | Authgear project endpoint. Used for OIDC discovery and JWKS. |
| `clientID` | `string` | | — | If set, the verifier also asserts the token's `client_id` claim. |
| `global` | `boolean` | | `false` | Register the guard as a global `APP_GUARD` so all routes are protected. |
| `jwksCacheMaxAge` | `number` | | — | JWKS cache max age in milliseconds. |
| `clockToleranceSeconds` | `number` | | `0` | Leeway in seconds for `exp`/`iat` checks. |

You can also inject `AuthgearTokenService` to verify a token outside of the guard.

***

### Next Steps

* [JWT access token reference](../../reference/tokens/jwt-access-token.md) — the full list of claims
* [Validate JWT in your backend](jwt.md) — the framework-agnostic approach
* [`@authgear/nestjs` documentation & API reference](https://authgear.github.io/authgear-sdk-nestjs/)
