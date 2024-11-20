---
description: Authentication for Next.js app with Authgear
---

# Next.js

In this guide, you'll learn how to implement authentication for the [Next.js](https://nextjs.org/) application, a popular React-based framework for JavaScript, and [Authgear](https://www.authgear.com/) as the OIDC provider. The source code can be found on [GitHub](https://github.com/authgear/authgear-example-nextjs).

### Learning objectives

You will learn the following throughout the article:

* How to add user login, sign-up, and logout to Next.js Applications.
* How to create a middleware to protect Next.js application pages.

### Implementing authentication in a Next.js web app

In the [demo application](https://github.com/authgear/authgear-example-nextjs), the Next.js app is integrated with Authgear, and the [NextAuth.js](https://next-auth.js.org/) client library is used for sending authentication requests as an **OpenID Connect middleware** from the app to Authgear.

### **Prerequisites**

Before you begin, you'll need the following:

* A **free Authgear account**. [Sign up](https://accounts.portal.authgear.com/signup) if you don't have one already.
* [Node.js](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm).
* Experience with [Next.js](https://nextjs.org/) framework and application development.

### Part 1: Configure Authgear

To use Authgear services, you’ll need to have an application set up in the Authgear [Dashboard](https://portal.authgear.com/). This setup allows users in Authgear to sign in to the Next.js application automatically once they are authenticated by Authgear.

#### Step 1: Configure an application

To set up the application, navigate to the [Authgear Portal UI](https://portal.authgear.com/) and select **Applications** on the left-hand navigation bar. Use the interactive selector to create a new **Authgear OIDC Client application** or select an existing application that represents the project you want to integrate with.

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Every application in Authgear is assigned an alphanumeric, unique client ID that your application code will use to call Authgear APIs through the NextAuth.js Client in the Next.js app. Record the generated Authgear `ISSUER` (for example, [https://example-auth.authgear.cloud](https://example-auth.authgear.cloud)), `CLIENT ID`, `CLIENT SECRET` from the output. You will use these values in the next step for the client app config.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

#### Step 2: Configure **Redirect URI**

An **Authorized Redirect URI** of your application is the URL that Authgear will redirect to after the user has authenticated for the **OpenID Connect middleware** to complete the authentication process. In our case, it will be a home page for our Next.js and it will run at [http://localhost:3000](http://localhost:3000).

Set the following [http://localhost:3000/api/auth/callback/authgear](http://localhost:3000/api/auth/callback/authgear) to the **Authorized Redirect URIs** field. If not set, users will not be returned to your application after they log in.

#### Step 3: Choose a Login method

After you created the **Authgear app**, you choose how users need to **authenticate on the login page**. From the **Authentication** tab, navigate to **Login Methods**, you can choose a **login method** from various options including, by email, mobile, or social, just using a username or the custom method you specify. For this demo, we choose the **Email+Passwordless** approach where our users are asked to register an account and log in by using their emails. They will receive a One-time password (OTP) to their emails and verify the code to use the app.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### Part 2: **Create the Next.js application**

You have two options here. You can either clone a [working example](https://github.com/authgear/authgear-example-nextjs) or build the app from scratch.

#### Clone the demo repository

If you want to run an already working application, you can clone the demo project from this [GitHub repository](https://github.com/authgear/authgear-example-nextjs) using the following command.

```bash
git clone https://github.com/authgear/authgear-example-nextjs.git
```

Since you've cloned a working repo, you don't need to follow the next section. If you'd like to understand more about what was done in the demo application, feel free to read them.

Either way, continue configuring the [Environment variables](https://fusionauth.io/blog/2023/04/26/nextjs-single-sign-on#environment-variables) to proceed with this tutorial.

#### Step 1: Create your own Next.js application

If you want to create your own application instead of using our demo project, you can create a new Next.js application by running the command below.

```bash
npx create-next-app authgear-example-nextjs
```

The `create-next-app` wizard will ask you a few questions on how to set up your application. Answer them accordingly.

#### Step 2: Installing NextAuth.js

Now that you have the application running, let's implement the authentication process by using [NextAuth.js](https://next-auth.js.org/). We recommend you look also at the [NextAuth.js Getting Start guide](https://next-auth.js.org/getting-started/example) for the most up-to-date instructions. First, install NextAuth.js:

```bash
npm install next-auth
```

Now, you need to create a file named exactly like `[...nextauth].js` in `src/pages/api/auth`. First, make the directory and then create a file named `[...nextauth].js` in that directory.

Next, you'll configure a [custom provider](https://next-auth.js.org/configuration/providers/oauth#using-a-custom-provider) for Authgear. Doing so ensures every request to the `/api/auth/*` path is handled by NextAuth.js.

In the following code, we made a few config:

1. We've added [`https://authgear.com/scopes/full-userinfo`](https://authgear.com/scopes/full-userinfo)as the scope to make Authgear return all user profiles
2. We have made `id` and `name` in the Next Auth session (under `callbacks`). You can also add other attributes such as `email`, `phone_number`, and `preferred_username` to the session as well.

```jsx
import NextAuth from "next-auth"

export const authOptions = {
    providers: [
        {
            id: "authgear",
            name: "Authgear",
            type: "oauth",
            issuer: process.env.AUTHGEAR_ISSUER,
            clientId: process.env.AUTHGEAR_CLIENT_ID,
            clientSecret: process.env.AUTHGEAR_CLIENT_SECRET,
            wellKnown: `${process.env.AUTHGEAR_ISSUER}/.well-known/openid-configuration`,
            authorization: { params: { scope: "openid offline_access https://authgear.com/scopes/full-userinfo" } },
            client: {
                token_endpoint_auth_method: "client_secret_post",
            },
            profile(profile) {
              return {
                id: profile.sub,
              }
            },
          }
    ],
    callbacks: {
        async jwt({ token, account, profile }) {
            if (account) {
                token.accessToken = account.access_token

                token.id = profile.sub
                token.name = profile.name
            }
            return token
        },

        async session({ session, token, user }) {
            session.accessToken = token.accessToken

            session.user.id = token.id
            session.user.name = token.name

            return session
        }
    },
}

export default NextAuth(authOptions)
```

#### Step 3: Exposing session state

To allow components to check whether the current user is logged in, change `src/pages/_app.js` to have your application rendered inside a `<SessionProvider>` context, as shown below.

```jsx
import '@/styles/globals.css'
import { SessionProvider } from "next-auth/react"

export default function App({
    Component,
    pageProps: {session, ...pageProps},
}) {
    return (
        <SessionProvider session={session}>
            <Component {...pageProps} />
        </SessionProvider>
    );
}
```

This will make the[ useSession()](https://next-auth.js.org/getting-started/client#usesession) React Hook accessible to your entire application. Now, create a component that will either render a "Log in" or "Log out" button, depending on the session state, in a `src/components/login-button.jsx` file.

```jsx
import { useSession, signIn, signOut } from "next-auth/react"

export default function Component() {
    const { data: session } = useSession()
    if (session) {
        return (
            <>
                Status: Logged in as {session.user.id} <br />
                <button onClick={() => signOut()}>Log out</button>
            </>
        )
    }
    return (
        <>
            Status: Not logged in <br />
            <button onClick={() => signIn()}>Log in</button>
        </>
    )
}
```

Then, you change your home component located at `src/pages/index.js` to include the `<LoginButton />` component inside `<main>`.

```jsx
import { useSession, signIn, signOut } from "next-auth/react"

export default function Component() {
    const { data: session } = useSession()
    if (session) {
        return (
            <>
                Status: Logged in as {session.user.id} <br />
                <button onClick={() => signOut()}>Log out</button>
            </>
        )
    }
    return (
        <>
            Status: Not logged in <br />
            <button onClick={() => signIn()}>Log in</button>
        </>
    )
}
```

#### Step 4: Set environment variables

In the root directory of your project, add the file `.env.local` with the following environment variables:

```jsx
AUTHGEAR_ISSUER={your-authgear-app-endpoint}
AUTHGEAR_CLIENT_ID={your-client-id}
AUTHGEAR_CLIENT_SECRET={your-client-secret}
```

replace with Authgear app settings values from **Part1** such as `Issuer`, `ClientId`, `ClientSecret`.

#### Step 5: Testing

Start the HTTP server by running the following command.

```bash
npm run dev
```

Browse to [localhost:3000](http://localhost:3000/). If the installation went successful, you should see the Login page.

<figure><img src="../../.gitbook/assets/Untitled (22).png" alt=""><figcaption></figcaption></figure>

Click the "Log in" button to be taken to a page with a "Sign in with Authgear" button.

<figure><img src="../../.gitbook/assets/Untitled (23).png" alt=""><figcaption></figcaption></figure>

After clicking it, you should be redirected to your Authgear login screen.

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Your users can sign-up and login to your application through a page hosted by Authgear, which provides them with a secure, standards-based login experience that you can customize with your own branding and various authentication methods, such as [social logins](https://www.authgear.com/features/social-login), [passwordless](https://www.authgear.com/features/passwordless-authentication), [biometrics logins](https://www.authgear.com/features/biometric-authentication), [one-time-password (OTP)](https://www.authgear.com/features/whatsapp-otp) with SMS/WhatsApp, and multi-factor authentication (MFA).

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

After you have authenticated with a one-time password sent to your email, you'll arrive back at your Next.js application home screen, with your email address displayed and a "Log out" button.

#### Next steps

This tutorial showed how to quickly implement an end-to-end OpenID Connect flow in Next.js with Authgear. Only simple code is needed, after which protected views are secured with built-in UI login pages.
