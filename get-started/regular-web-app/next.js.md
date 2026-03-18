---
description: Authentication for Next.js app with Authgear
---

# Next.js

Integrate Authgear authentication into a Next.js App Router application using the `@authgear/nextjs` SDK. You will set up login, logout, display user info, and protect a server-side API route.

A complete example application is available at [authgear/authgear-example-nextjs](https://github.com/authgear/authgear-example-nextjs).

**What you will build:**

* A home page that shows a Login button when unauthenticated, and the user's identity + Logout button when authenticated
* A protected API route (`/api/me`) that returns the current user's info

***

### Setting Up Your Application in Authgear

#### Step 1: Create an Application in the Portal

1. Sign in to the [Authgear Portal](https://portal.authgear.com/)
2. Select or create a Project
3. Navigate to **Applications** in the left menu
4. Click **⊕ Add Application**
5. Enter an application name and select **OIDC/SAML Client Application** as the application type
6. Click **Save**

#### Step 2: Configure the Application

1. Under **OAuth 2.0**, find the **Authorized Redirect URIs** field
2. Add `http://localhost:3000/api/auth/callback`
3. Note down your **Client ID**, **Client Secret**, and **Endpoint** (e.g. `https://your-project.authgear.cloud`) — you will need these shortly
4. Click **Save**

***

### Building Your Next.js Application

#### Step 1: Create a Next.js Project

```bash
npx create-next-app@latest my-app --typescript --tailwind --app
cd my-app
```

#### Step 2: Install the Authgear SDK

```bash
npm install @authgear/nextjs
```

#### Step 3: Configure Environment Variables

Create `.env.local` with the following content:

```bash
AUTHGEAR_ENDPOINT=https://your-project.authgear.cloud
AUTHGEAR_CLIENT_ID=your-client-id
AUTHGEAR_CLIENT_SECRET=your-client-secret
AUTHGEAR_REDIRECT_URI=http://localhost:3000/api/auth/callback
SESSION_SECRET=a-random-string-of-at-least-32-characters
```

{% hint style="info" %}
`SESSION_SECRET` encrypts the session cookie stored in the browser. It must be at least 32 characters. Use a random string generator to create one.
{% endhint %}

#### Step 4: Create the Shared Authgear Config

Create `src/lib/authgear.ts`. This file holds your Authgear configuration and is imported by both server-side and client-side code:

```typescript
// src/lib/authgear.ts
import type { AuthgearConfig } from "@authgear/nextjs";

export const authgearConfig: AuthgearConfig = {
  endpoint: process.env.AUTHGEAR_ENDPOINT!,
  clientID: process.env.AUTHGEAR_CLIENT_ID!,
  clientSecret: process.env.AUTHGEAR_CLIENT_SECRET,
  redirectURI: process.env.AUTHGEAR_REDIRECT_URI!,
  sessionSecret: process.env.SESSION_SECRET!,
};
```

#### Step 5: Add the OAuth Route Handler

Create `src/app/api/auth/[...authgear]/route.ts`. This catch-all route handles all Authgear auth endpoints automatically:

```typescript
// src/app/api/auth/[...authgear]/route.ts
import { createAuthgearHandlers } from "@authgear/nextjs";
import { authgearConfig } from "@/lib/authgear";

export const { GET, POST } = createAuthgearHandlers(authgearConfig);
```

`createAuthgearHandlers` registers the following routes for you:

| Method | Path                 | Purpose                                              |
| ------ | -------------------- | ---------------------------------------------------- |
| `GET`  | `/api/auth/login`    | Start the OAuth login flow                           |
| `GET`  | `/api/auth/callback` | Handle the OAuth callback and set the session cookie |
| `GET`  | `/api/auth/logout`   | Clear the session and revoke tokens                  |
| `POST` | `/api/auth/refresh`  | Refresh an expired access token                      |
| `GET`  | `/api/auth/userinfo` | Return the current user's info                       |

#### Step 6: Add AuthgearProvider to Your Layout

`AuthgearProvider` makes the user's session state available to all Client Components via React context. Because it uses browser APIs, it must be a Client Component.

Create `src/app/providers.tsx`:

```typescript
// src/app/providers.tsx
"use client";

import { AuthgearProvider } from "@authgear/nextjs/client";

export default function Providers({ children }: { children: React.ReactNode }) {
  return <AuthgearProvider>{children}</AuthgearProvider>;
}
```

Then wrap your root layout in `src/app/layout.tsx`:

```typescript
// src/app/layout.tsx
import type { Metadata } from "next";
import { Geist, Geist_Mono } from "next/font/google";
import "./globals.css";
import Providers from "./providers";

const geistSans = Geist({ variable: "--font-geist-sans", subsets: ["latin"] });
const geistMono = Geist_Mono({ variable: "--font-geist-mono", subsets: ["latin"] });

export const metadata: Metadata = {
  title: "Next.js + Authgear",
  description: "Example app demonstrating Authgear authentication with Next.js",
};

export default function RootLayout({
  children,
}: Readonly<{ children: React.ReactNode }>) {
  return (
    <html lang="en">
      <body className={`${geistSans.variable} ${geistMono.variable} antialiased`}>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

On mount, `AuthgearProvider` fetches `/api/auth/userinfo` to check for an existing session. The result is available via the `useAuthgear()` hook.

#### Step 7: Implement the Home Page

Replace `src/app/page.tsx`. The page is a Client Component so it can use `useAuthgear()` to read session state and react to changes:

```typescript
// src/app/page.tsx
"use client";

import { useState } from "react";
import { useAuthgear, SignInButton, SignOutButton } from "@authgear/nextjs/client";

export default function Home() {
  const { isAuthenticated, user } = useAuthgear();
  const [apiResult, setApiResult] = useState<string | null>(null);

  async function testProtectedApi() {
    const res = await fetch("/api/me");
    const data = await res.json();
    setApiResult(JSON.stringify(data, null, 2));
  }

  return (
    <main className="flex min-h-screen flex-col items-center justify-center gap-6">
      <h1 className="text-3xl font-bold">Next.js + Authgear</h1>

      {isAuthenticated ? (
        <>
          <p className="text-gray-600">
            Logged in as: <span className="font-mono">{user?.sub}</span>
          </p>
          {(user?.email ?? user?.phoneNumber) && (
            <p className="text-gray-600">
              {user?.email ?? user?.phoneNumber}
            </p>
          )}
          <button
            onClick={testProtectedApi}
            className="rounded-md bg-green-600 px-6 py-2 text-white hover:bg-green-700"
          >
            Test Protected API
          </button>
          {apiResult && (
            <pre className="rounded-md bg-gray-100 dark:bg-gray-800 p-4 text-sm">{apiResult}</pre>
          )}
          <SignOutButton className="rounded-md bg-red-600 px-6 py-2 text-white hover:bg-red-700">
            Logout
          </SignOutButton>
        </>
      ) : (
        <SignInButton className="rounded-md bg-blue-600 px-6 py-2 text-white hover:bg-blue-700">
          Login
        </SignInButton>
      )}
    </main>
  );
}
```

**Key hooks and components:**

* `useAuthgear()` — returns `{ isAuthenticated, user, state, isLoaded, signIn, signOut }`. `user` is a `UserInfo` object with `sub`, `email`, `phoneNumber`, and other profile fields.
* `<SignInButton>` — navigates to `/api/auth/login` on click, starting the OAuth flow.
* `<SignOutButton>` — navigates to `/api/auth/logout` on click, clearing the session.

#### Step 8: Create a Protected API Route

Server-side route handlers can verify authentication using `currentUser()` from `@authgear/nextjs/server`. It reads the encrypted session cookie and returns the current user, or `null` if not authenticated.

Create `src/app/api/me/route.ts`:

```typescript
// src/app/api/me/route.ts
import { currentUser } from "@authgear/nextjs/server";
import { authgearConfig } from "@/lib/authgear";
import { NextResponse } from "next/server";

export async function GET() {
  const user = await currentUser(authgearConfig);

  if (!user) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  return NextResponse.json({ user });
}
```

`currentUser()` automatically refreshes the access token if expired before fetching user info from the Authgear userinfo endpoint.

***

### Running the Application

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

***

### Testing the Integration

#### Login flow

1. Visit `http://localhost:3000` — you should see a **Login** button
2. Click **Login** — you are redirected to the Authgear hosted login page
3. Sign in with your credentials
4. You are redirected back to the home page, now showing your user ID and email/phone number

#### Logout flow

1. Click **Logout** — the session is cleared and you are returned to the login screen

#### Protected API

1. After logging in, click **Test Protected API**
2. The page calls `GET /api/me`, which verifies your session and returns your user info:

```json
{
  "user": {
    "sub": "...",
    "email": "...",
    "emailVerified": true
  }
}
```

3. To verify the route is protected, open an incognito window and run:

```bash
curl http://localhost:3000/api/me
```

Expected response with status `401`:

```json
{ "error": "Unauthorized" }
```

***

### Project Structure

```
src/
├── app/
│   ├── api/
│   │   ├── auth/
│   │   │   └── [...authgear]/
│   │   │       └── route.ts      # OAuth route handler
│   │   └── me/
│   │       └── route.ts          # Protected API route
│   ├── layout.tsx                # Root layout with AuthgearProvider
│   ├── page.tsx                  # Home page with login/logout UI
│   └── providers.tsx             # Client component wrapping AuthgearProvider
└── lib/
    └── authgear.ts               # Shared Authgear config
```

***

### Additional Topics

#### Reading the Session in Server Components

Use `auth()` to read the raw session (state + tokens) without making a network request:

```typescript
import { auth } from "@authgear/nextjs/server";
import { authgearConfig } from "@/lib/authgear";

export default async function MyServerComponent() {
  const session = await auth(authgearConfig);

  if (session.state !== "AUTHENTICATED") {
    return <p>Not logged in</p>;
  }

  return <p>Access token expires at: {session.expiresAt}</p>;
}
```

Use `currentUser()` when you need the full user profile — it makes an extra request to the userinfo endpoint and handles token refresh automatically.

#### Verifying a Bearer Token in an External API

If you have a separate API server that receives Authgear-issued access tokens (e.g. from a mobile app), use `verifyAccessToken()` to validate them:

```typescript
import { verifyAccessToken } from "@authgear/nextjs/server";
import { authgearConfig } from "@/lib/authgear";
import { NextRequest, NextResponse } from "next/server";

export async function GET(request: NextRequest) {
  const authHeader = request.headers.get("authorization");
  if (!authHeader?.startsWith("Bearer ")) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  const token = authHeader.slice(7);

  try {
    const payload = await verifyAccessToken(token, authgearConfig);
    return NextResponse.json({ sub: payload.sub });
  } catch {
    return NextResponse.json({ error: "Invalid token" }, { status: 401 });
  }
}
```

{% hint style="info" %}
`verifyAccessToken` validates the JWT signature and expiry locally using the Authgear JWKS endpoint — no session cookie is involved.
{% endhint %}

#### Checking Session State

`useAuthgear()` returns `isLoaded` to indicate whether the initial session check has completed. Use it to avoid a flash of unauthenticated UI:

```typescript
const { isAuthenticated, isLoaded } = useAuthgear();

if (!isLoaded) return <p>Loading...</p>;
```

### Next Steps

* [Protect additional pages with middleware](https://nextjs.org/docs/app/building-your-application/routing/middleware)
* [Backend API integration guide](https://docs.authgear.com/get-started/backend-api)
* [`@authgear/nextjs` API reference](https://authgear.github.io/authgear-sdk-nextjs/)
