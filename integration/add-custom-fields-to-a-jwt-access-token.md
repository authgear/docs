---
description: Learn how to add custom attributes to a JWT Access Token using Authgear
---

# Add custom fields to a JWT Access Token

JWTs (JSON Web Tokens) are a common method for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. With Authgear, it is straightforward to add custom fields to your JWT access tokens.

This how-to guide will walk you through the process of **adding custom fields such as** [**User Profiles**](https://docs.authgear.com/integrate/user-profile) **attributes to a JWT access token** payload using Authgear and Javascript [Hooks](https://docs.authgear.com/integrate/events-hooks/denohooks).

Here's an example of the [fields in the JWT Access Tokens by default](../reference/tokens/jwt-access-token.md) and an explanation of their values.

{% hint style="info" %}
You can also [add custom attributes](https://docs.authgear.com/integrate/user-profile#add-new-attributes) to [User Profiles](https://docs.authgear.com/integrate/user-profile) on the Authegear Portal.
{% endhint %}

### Prerequisites

* **An Authgear account:** You need an Authgear account to follow this guide. If you don't have one, you can [create it for free](https://accounts.portal.authgear.com/signup) on the Authgear website.
* **A Registered App:** You need a [registered application](https://docs.authgear.com/get-started/website#setup-application-in-authgear) (client) in Authgear.

### Enable Access Token for your App

Make sure the option **Issue JWT as access token** is enabled in your **Application** settings in the Portal.

1. Log into your [Authgear account](https://portal.authgear.com/?_ga=2.25390521.563520449.1688969336-1174359617.1686657394).
2. Navigate to the **Applications** tab and choose the existing App.
3. On the **App Configuration** dashboard, locate the "**Access token**" section.
4. Make the toggle **Issue JWT as access token** switch on.

<figure><img src="../.gitbook/assets/image (28).png" alt="" width="537"><figcaption></figcaption></figure>

### Create a new Event Hook

With the use of Hooks, Authgear provides flexibility for adding custom logic to your authentication pipeline. You can create a Hook which is triggered any of these [Events ](https://docs.authgear.com/integrate/events-hooks/event-list)about to occur. For example, `oidc.jwt.pre_create` the event happens just before issuing the JWT access token and it can be used to put extra information into the token.

1. Navigate to your Authgear Dashboard's **Advanced**->**Hooks** section.
2. **Add** a new **Blocking Event**.
3. Choose the Block Hook **Type** as the _TypeScript_ and set the Event option to _JWT access token pre-create_. You will write a new Typescript function from scratch.

<figure><img src="../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

4. Click on **Edit Script** under the **Config** option.
5. Copy and paste the following into the editor:

```typescript
import { EventOIDCJWTPreCreate, HookResponse } from "https://deno.land/x/authgear_deno_hook@v1.0.0/mod.ts";

export default async function(e: EventOIDCJWTPreCreate): Promise<HookResponse> {
  return {
    mutations:{
      jwt: {
        payload:{
          ...e.payload.jwt.payload,
          standard_attributes: e.payload.user.standard_attributes,
          custom_attributes: e.payload.user.custom_attributes
        }
      }
    },
    is_allowed: true
  };
}
```

6. Click on **Finish Editing**.
7. Back to the **Hooks** page from the navigation bar and click on the **Save** button at the top of the page.

In the above code, we are importing the necessary modules such as `HookResponse` and `EventOIDCJWTPreCreate` which are types from the Authgear [Deno](https://deno.land/) hook [Typescript library](https://deno.land/x/authgear_deno_hook). We modify the JWT payload by adding [Standard Attributes](https://docs.authgear.com/integrate/user-profile#standard-attributes)(`e.payload.user.standard_attributes`) and [Custom Attributes](https://docs.authgear.com/integrate/user-profile#custom-attributes)(`e.payload.user.custom_attributes`) of the user.

### Verify the Custom Field in a JWT token

There are two ways to test it:

* You can do this by [decoding the JWT token](https://docs.authgear.com/get-started/backend-integration/jwt) on your application server side using a JWT decoder and inspecting the payload.
* If you created the application type **OIDC Client Application,** you need to follow the steps below. Expand it to see instructions.

<details>

<summary>Verify the custom field in the JWT Token with <strong>OIDC Client Application</strong></summary>

This part explains how to retrieve an access token using **OpenID App** Endpoints and check if newly added custom attributes are in place in the JWT Access token.

**Prerequisites**

* Make sure that you have a registered app type of **OIDC Client Application** in Authgear Portal.

**Step 1: Obtain the necessary parameters**

Open your **OpenID Auth App** configuration, and find **Client ID**, **Client Secret**, and check **Authorization**, and **Token** endpoints. You will use them in the next steps.

<img src="../.gitbook/assets/image (10).png" alt="" data-size="original"> <img src="../.gitbook/assets/image (6).png" alt="" data-size="original">

**Step 2: Construct the authorization endpoint URL**

The URL for this endpoint is usually provided by the authorization server and includes parameters specifying the requested `scope`, `client_id`, and response\_type. Here's an example URL for the authorization endpoint:

```
https://<YOUR_AUTHGEAR_ENDPOINT>/oauth2/authorize?client_id={YOUR_CLIENT_ID}&response_type=code&scope=openid
```

Replace `<YOUR_AUTHGEAR_ENDPOINT>` with your Authgear server's domain, `YOUR_CLIENT_ID` with your application's Client ID from **OpenID App.**

**Step 3: Redirect the user to the authorization endpoint**

Next, you need to redirect the user to the authorization endpoint. You can just put the URL in your browser and log in with a user credential you are interested to retrieve an access token for. After successful authentication and consent, the Authgear will redirect the user back to your specified redirect URI, including an **authorization code** as a query parameter. You will need the code in the next step

<img src="../.gitbook/assets/image (12).png" alt="" data-size="original">

**Step 4: Obtain an access token**

You need to make a request to the **OpenID App's Token endpoint** to exchange the authorization code we retrieved in the previous step for an access token.

* The token endpoint URL is usually something like `https://<YOUR_AUTHGEAR_ENDPOINT>/oauth2/token`.
* Include parameters such as `grant_type=authorization_code`, `code=AUTHORIZATION_CODE`, `client_id=YOUR_CLIENT_ID`, `client_secret=YOUR_CLIENT_SECRET`, and `redirect_uri=YOUR_REDIRECT_URI`.
* Make a POST request to the token endpoint to obtain the access token.

```bash
curl --request POST \
  --url 'https://<YOUR_AUTHGEAR_ENDPOINT>/oauth2/token' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data grant_type=authorization_code \
  --data code={YOUR_AUTHORIZATION_CODE} \
  --data redirect_uri={YOUR_REDIRECT_URI} \
  --data 'client_id={YOUR_CLIENT_ID}' \
  --data client_secret={YOUR_CLIENT_SECRET} \
  --data scope=openid
```

**Step 5: Verify custom attributes in the access token**

Finally, we can debug the access token using the [JWT Debugger tool](https://jwt.io/) to see if the custom field and value we added previously are there inside the JWT payload.

<img src="../.gitbook/assets/image (17).png" alt="" data-size="original">

\\

</details>
