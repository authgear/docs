---
description: >-
  Guide on using Authgear together with Supabase database to deliver a secure
  and scalable application
---

# Authgear and Supabase

User authentication is a foundational part of any modern web application but getting it right is notoriously difficult. That’s where Authgear comes in. Authgear handles user registration, login flows, session management, and advanced security features like biometric login, 2FA, and social sign-ins out of the box.

Pairing Authgear with Supabase's Postgres database and powerful Row-Level Security (RLS) gives you a secure, scalable foundation for building user-specific applications.

In this guide, you’ll learn how to integrate Authgear and Supabase using React.js to build a basic CRUD application.

<figure><img src="../../.gitbook/assets/image (58).png" alt="Simple instrument tracking app where users can manage their personal collection after logging in with Authgear"><figcaption><p>Simple instrument tracking app where users can manage their personal collection after logging in with Authgear</p></figcaption></figure>

We will walkthrough how to **authenticate users with Authgear**, exchange tokens via a **Supabase Edge Function**, apply **RLS policies** based on Authgear-issued JWTs, and connect it all in a modern React frontend using Vite.

```
┌─────────────┐          ┌──────────────┐          ┌──────────────┐
│   Browser   │          │   Authgear   │          │   Supabase   │
│  (React App)│◄────────►│  (Auth IdP)  │          │  (Database)  │
└──────┬──────┘          └──────────────┘          └──────┬───────┘
       │                                                  │
       │ 1. Login with Authgear                           │
       │ 2. Get Authgear JWT (access token)               │
       │ 3. Call Supabase with Authgear JWT (request data)│
       └──────────────────────────────────────────────────►
       │                                                   │
       │ 4. Exchange JWT (Edge Function)                   │
       │ 5. Return Supabase JWT                            │
       │◄──────────────────────────────────────────────────
       │                                                   │
       │ 6. Access data with Supabase JWT (enforce RLS)    │
       └──────────────────────────────────────────────────►

```

## **Full example code**

The full example app source code is available on GitHub and can be used as a starting point for your own project: [https://github.com/authgear/authgear-example-supabase](https://github.com/authgear/authgear-example-supabase)

## What you need

* **Authgear account & project** – [Sign up for Authgear](https://portal.authgear.com/).
* **Supabase account & project** – [Sign up on supabase.com](https://supabase.com/)

## Step 1: Set up an Authgear project

First, configure Authgear to handle authentication for your app.

1. **Create an Authgear Application:** Log in to the Authgear Portal and create a new application. Choose **Single Page Application** as the application type.
2. **Configure OAuth Redirects:** In your Authgear app settings, set the redirect URIs:
   * **Redirect URI**: `http://localhost:5173/auth-redirect` – this is where Authgear will redirect the browser after a successful login (our React app will handle this route).
   * **Post-Logout Redirect URI:** `http://localhost:5173/` – where to navigate after users log out.
3. **Note Your Authgear Credentials:** After creation, note down:
   * The **Authgear Endpoint** for your app (it will look like `https://<your-project>.authgear.cloud`).
   * The **Client ID** of your Authgear application.

We will need these values when initializing Authgear in our React app and in the Supabase function.

## Step 2: Set up a Supabase project

Next, prepare your Supabase backend.

1. **Create a Supabase Project:** Log in to Supabase Dashboard, create a new project (choose an organization, project name, database password, and region).&#x20;
2. **Retrieve Project Keys:** In your Supabase project, go to **Project Settings → API → API keys**. Copy the **Publishable Key** and the **URL** of your project (e.g., `https://xyzcompany.supabase.co`).
3.  **Get the JWT Secret:** Still in **Project settings**, navigate to **JWT Keys** and find the **Legacy JWT Secret** for your project. This is a secret key that Supabase uses to sign and verify JWTs for RLS. Copy the JWT secret value – we will use it in our Edge Function config.\


    <figure><img src="../../.gitbook/assets/image (59).png" alt=""><figcaption><p>Copy the Legacy JWT Secret from the JWT Keys page</p></figcaption></figure>


4. _(Optional)_ **Install Supabase CLI:** If you plan to use the CLI to deploy the Edge Function, install it by following [Supabase’s instructions](https://supabase.com/docs/guides/local-development/cli/getting-started). Alternatively, you can deploy the function via the Dashboard UI.

## Step 3: Create an Edge Function to exchange JWTs

Now we set up the critical piece: a Supabase Edge Function that will accept Authgear’s JWT and return a new JWT signed with Supabase’s secret.&#x20;

1. Navigate to "Edge Functions" -> "Secrets" and add the two secrets:
   1. `AUTHGEAR_ENDPOINT` = your Authgear app endpoint (e.g. `https://myapp.authgear.cloud`).
   2. `SB_JWT_SECRET` = your Supabase JWT secret (from Step 2 above).
2. Deploy the Function
   1. Navigate to "Edge Functions" -> "Functions" in Supabase web UI. Copy-paste the following code and deploy it there. Name the function `exchange-jwt`.
   2. Alternatively, you can use Supabase CLI to build and deploy the Edge Function from your local machine to your Supabase project.

```typescript
// supabase/functions/exchange-jwt/index.ts

import "jsr:@supabase/functions-js/edge-runtime.d.ts"
import jwt from "npm:jsonwebtoken";
import jwks from "npm:jwks-rsa";
import * as jose from "https://deno.land/x/jose@v4.14.4/index.ts";

// CORS headers for cross-origin requests
const corsHeaders = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Headers": "Authorization, Content-Type",
  "Access-Control-Allow-Methods": "GET",
  "Access-Control-Max-Age": "3600"
};

// Read environment variables
const authgearEndpoint = Deno.env.get("AUTHGEAR_ENDPOINT");
const supabaseJwtSecret = Deno.env.get("SB_JWT_SECRET");
if (!authgearEndpoint || !supabaseJwtSecret) throw new Error("Missing env vars");

// Get JWKS URI from Authgear's OpenID configuration
async function getJwksUri(): Promise<string> {
  const configUrl = `${authgearEndpoint}/.well-known/openid-configuration`;
  const response = await fetch(configUrl);
  const config = await response.json();
  return config.jwks_uri;
}

// Extract Bearer token from Authorization header
function extractToken(req: Request): string | null {
  const authHeader = req.headers.get("Authorization");
  if (!authHeader) return null;
  
  const parts = authHeader.split(" ");
  if (parts.length !== 2 || parts[0] !== "Bearer") return null;
  
  return parts[1];
}

// Verify JWT token with Authgear's public key
async function verifyToken(token: string): Promise<any> {
  // Decode token to get key ID
  const decoded = jwt.decode(token, { complete: true }) as any;
  if (!decoded?.header?.kid) {
    throw new Error("Invalid token: missing key ID");
  }

  // Get JWKS URI and create JWKS client
  const jwksUri = await getJwksUri();
  const jwksClient = jwks({ jwksUri });

  // Get the signing key
  const key = await jwksClient.getSigningKey(decoded.header.kid);
  const signingKey = key.getPublicKey();

  // Verify the token
  const options = {
    algorithms: ["RS256"],
    issuer: authgearEndpoint,
  };

  return jwt.verify(token, signingKey, options);
}

// Sign a new JWT with Supabase secret
async function signSupabaseJwt(payload: any): Promise<string> {
  payload.role = "authenticated"; // Required by Supabase

  // Add or modify any other claims you need for RLS policies
  // payload.some_claim = "some claim";

  // Sign with Supabase JWT secret
  const supabaseSecret = new TextEncoder().encode(supabaseJwtSecret);

  const supabaseJwt = await new jose.SignJWT(payload)
    .setProtectedHeader({ alg: "HS256", typ: "JWT" })
    .setIssuer("supabase")
    .setIssuedAt(payload.iat)
    .setExpirationTime(payload.exp || "")
    .sign(supabaseSecret);

  return supabaseJwt;
}

// Main function handler
Deno.serve(async (req: Request) => {
  // Handle CORS preflight requests
  if (req.method === "OPTIONS") {
    return new Response("ok", { headers: corsHeaders });
  }

  try {
    // Extract token from request
    const token = extractToken(req);
    if (!token) {
      return new Response(
        JSON.stringify({ error: "Missing or invalid Authorization header" }),
        { 
          headers: { ...corsHeaders, "Content-Type": "application/json" },
          status: 401 
        }
      );
    }
    // Verify the token
    const verified = await verifyToken(token);
    // Sign a new JWT with Supabase secret
    const supabaseJwt = await signSupabaseJwt(verified);
    return new Response(
      JSON.stringify({ supabaseJwt }),
      {
        headers: { ...corsHeaders, "Content-Type": "application/json" },
        status: 200,
      }
    );
  } catch (error) {
    console.error("Token verification failed:", error.message);
    return new Response(
      JSON.stringify({ error: "Token verification failed" }),
      {
        headers: { ...corsHeaders, "Content-Type": "application/json" },
        status: 401,
      }
    );
  }
});

```

Once deployed, your function is accessible at:&#x20;

```
${SUPABASE_URL}/functions/v1/exchange-jwt
```

## Step 4: Create the database table and RLS policies

Now let’s set up the database table and RLS rules in Supabase. We’ll create a table (`instruments`) to store items owned by users, and use RLS to ensure each user can only manipulate their own rows.

Open the SQL editor in your Supabase project and run the following SQL commands:

```sql
-- 1. Helper function to get the current Authgear user ID from the JWT
CREATE OR REPLACE FUNCTION current_user_id()
RETURNS TEXT AS $$
  SELECT auth.jwt() ->> 'sub';
$$ LANGUAGE SQL STABLE;
```

We created a SQL function `current_user_id()` that returns the JWT’s `sub` claim (subject) from the current request’s JWT. Supabase’s Postgres has an `auth.jwt()` function that exposes the JWT claims of the requester; `auth.jwt() ->> 'sub'` extracts the `sub` field as text.

```sql
-- 2. Instruments table: each row has a user_id to identify the owner
CREATE TABLE instruments (
  id BIGSERIAL PRIMARY KEY,
  created_at TIMESTAMPTZ DEFAULT NOW() NOT NULL,
  name TEXT NOT NULL,
  user_id TEXT NOT NULL   -- store Authgear user’s ID (sub)
);
 
-- 3. Enable Row Level Security on the table
ALTER TABLE instruments ENABLE ROW LEVEL SECURITY;

```

* The `instruments` table has a `user_id` column which will store the Authgear user’s ID for each instrument.
* We enabled RLS on the table, which means by default no rows can be accessed unless allowed by a policy.

```sql
-- 4. RLS Policies:
-- Allow each authenticated user to SELECT rows where user_id = their own ID
CREATE POLICY "Users can read their own instruments"
ON instruments
FOR SELECT
TO authenticated
USING (user_id = current_user_id());
 
-- Allow INSERTs only if setting user_id to their own ID
CREATE POLICY "Users can insert their own instruments"
ON instruments
FOR INSERT
TO authenticated
WITH CHECK (user_id = current_user_id());
 
-- Allow UPDATE on rows they own
CREATE POLICY "Users can update their own instruments"
ON instruments
FOR UPDATE
TO authenticated
USING (user_id = current_user_id())
WITH CHECK (user_id = current_user_id());
 
-- Allow DELETE on rows they own
CREATE POLICY "Users can delete their own instruments"
ON instruments
FOR DELETE
TO authenticated
USING (user_id = current_user_id());
```

We then defined four policies, one for each CRUD operation (SELECT, INSERT, UPDATE, DELETE). Each policy is limited to the `authenticated` role and uses a condition requiring that the row’s `user_id` matches the user’s id from the JWT.

## Step 5. Build the React app with Supabase Client

Now for the frontend. We will create a React app using Vite, integrate Authgear’s SDK for authentication, and configure the Supabase client to use our token exchange flow automatically.

**Dependencies**

1. Scaffold a React App by `npm create vite@latest my-app -- --template react`
2. Follow the guide for build SPA in here: [react.md](../../get-started/single-page-app/react.md "mention") to setup a basic react app with Authgear authentication
3.  Install the Supabase library by&#x20;

    ```
    npm install @supabase/supabase-js
    ```
4. Configure the Environment Variables in `.env` and make the Supabase URL and Publishable key available. You `.env.local` should look like:

```
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=your-supabase-anon-key

VITE_AUTHGEAR_ENDPOINT=https://your-app.authgear.cloud
VITE_AUTHGEAR_CLIENT_ID=your-authgear-client-id
VITE_AUTHGEAR_REDIRECT_URL=http://localhost:5173/auth-redirect
VITE_AUTHGEAR_LOGOUT_REDIRECT_URL=http://localhost:5173/
```

**Configure the Supabase Client**

Now we can set up the Supabase JS client to use our Authgear token. Create a new file `my-app/src/lib/supabase.js` and add the following:

```javascript
// my-app/src/lib/supabase.js
import { createClient } from '@supabase/supabase-js';
import authgear from '@authgear/web';

const SUPABASE_URL = import.meta.env.VITE_SUPABASE_URL;
const SUPABASE_ANON_KEY = import.meta.env.VITE_SUPABASE_PUBLISHABLE_KEY;

// Optional: caches to avoid redundant token exchanges
let cachedSupabaseJwt = null;
let cachedAuthgearToken = null;

// Function to exchange Authgear token for Supabase token
async function exchangeToken(authgearToken) {
  // Return cached token if the Authgear token hasn’t changed
  if (cachedAuthgearToken === authgearToken && cachedSupabaseJwt) {
    return cachedSupabaseJwt;
  }
  // Call the Supabase Edge Function
  const res = await fetch(`${SUPABASE_URL}/functions/v1/exchange-jwt`, {
    headers: { Authorization: `Bearer ${authgearToken}` }
  });
  if (!res.ok) {
    throw new Error(`Token exchange failed with status ${res.status}`);
  }
  const { supabaseJwt } = await res.json();
  // Update cache and return the new JWT
  cachedAuthgearToken = authgearToken;
  cachedSupabaseJwt = supabaseJwt;
  return supabaseJwt;
}

// Create Supabase client with custom auth settings
export const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY, {
  auth: {
    autoRefreshToken: false,   // Don't use Supabase’s token refresh
    persistSession: false,     // Don't store Supabase session (Authgear will handle session)
  },
  accessToken: async () => {
    // This function runs before every Supabase request that needs auth
    await authgear.refreshAccessTokenIfNeeded();
    const authgearToken = authgear.accessToken;
    if (!authgearToken) return null;
    // Exchange Authgear token for a Supabase token:contentReference[oaicite:12]{index=12}:contentReference[oaicite:13]{index=13}
    return exchangeToken(authgearToken);
  },
});

```

In this configuration:

* We disable `autoRefreshToken` and `persistSession` in Supabase’s client so that Authgear is the source of truth for the session.
* We provide an `accessToken` async function. The Supabase Client will call this function every time it needs a JWT for an authenticated request. In our case, that is every query to our `instruments` table.
* In `accessToken()`, we first ensure Authgear’s token is fresh by calling `authgear.refreshAccessTokenIfNeeded()`.&#x20;
* We then get `authgear.accessToken` – this is the current JWT from Authgear’s SDK.
* If the token exists, we call our `exchangeToken()` helper, which calls the Edge Function’s endpoint with the Authgear token and receives a Supabase JWT in response.

**Build the CRUD functions**

After login, use the Supabase client to fetch and manipulate data. For example, to load the current user’s instruments, you can call:

{% code overflow="wrap" %}
```javascript
const { data, error } = await supabase.from('instruments').select('*');
```
{% endcode %}

To add a new instrument, call&#x20;

```javascript
supabase.from('instruments').insert({
    name: "<new intruments>",
    user_id: userInfo.sub,
});
```

Similarly, to update or delete, similarly call `.update(...)` or `.delete(...)` in the Supabase Client with appropriate filters

## 6. Run and test the app

We’re ready to test the whole system. Start the development server:

```bash
npm run dev
```

Open `http://localhost:5173` in your browser and test the integration.

## That's it

You’ve successfully integrated Authgear for authentication with Supabase for secure, per-user data access using Row-Level Security.

With this setup, you can take full advantage of Authgear’s powerful authentication features like social logins, 2FA, and secure session management, while leveraging Supabase’s scalable database and built-in RLS to keep user data isolated and protected. This token exchange approach makes it easy to use Authgear (or any other JWT-based provider) with Supabase.
