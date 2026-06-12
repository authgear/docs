# Link and Unlink Social/Enterprise Provider with the SDK

Let signed-in users connect a social or enterprise login provider to their account, and disconnect it later, from your own UI. This guide uses Google as the example provider, but the same code works for any OAuth provider you have configured.

{% hint style="info" %}
These SDK methods are currently available in the **web SDK (`@authgear/web`) only**. For other SDKs, use the [user-settings.md](../../customization/ui-customization/built-in-ui/user-settings.md "mention") page and users can manage their Social/Enterprise connection there.
{% endhint %}

### What you will build

A page in your app, such as an account details page, that:

1. Reads the signed-in user's linked identities.
2. Shows a **Link** button when the provider is not connected, or an **Unlink** button when it is.
3. Links the provider by redirecting to Google's consent screen, then back to your app.
4. Unlinks the provider by redirecting to an Authgear-hosted confirmation page, then back to your app.

Both actions use a browser redirect. The user leaves your app, completes the step on Google's or Authgear's page, and returns to a redirect URI you control. Your callback code calls a `finish` method to complete the action.

### Prerequisites

* The `@authgear/web` SDK installed and configured in your app. See [Getting Started](../../get-started/start-building.md).
* The user is **already signed in**. Linking and unlinking require an authenticated session. Both methods throw if no valid session exists.
* The OAuth provider is configured in the Authgear Portal under **Social / Enterprise Login**, with an alias. This guide uses the alias `google`.
* The redirect URI you pass to each method is registered under your application's **Redirect URIs** in the Portal.

The examples assume your app configures the SDK with `sessionType: "refresh_token"`.

### Step 1: Check whether the provider is linked

Call `fetchUserInfo()` to get the user's info, then inspect the `identities` array. Each OAuth identity has `type === IdentityType.OAuth` and an `oauthProviderAlias` that matches the alias you configured in the Portal.

```typescript
import authgear, { IdentityType } from "@authgear/web";

const PROVIDER_ALIAS = "google";

async function isProviderLinked() {
  const userInfo = await authgear.fetchUserInfo();
  return (userInfo.identities ?? []).some(
    (identity) =>
      identity.type === IdentityType.OAuth &&
      identity.oauthProviderAlias === PROVIDER_ALIAS
  );
}
```

The raw `identities` claim from the [UserInfo endpoint](../../reference/apis/oauth-2.0-and-openid-connect-oidc/userinfo.md) looks like this:

```json
{
  "https://authgear.com/claims/user/identities": [
    {
      "type": "login_id",
      "login_id_key": "email",
      "login_id_type": "email"
    },
    {
      "type": "oauth",
      "oauth_provider_type": "google",
      "oauth_provider_alias": "google"
    }
  ]
}
```

### Step 2: Show the Link or Unlink button

Render the button that matches the current state.

{% tabs %}
{% tab title="React" %}
```tsx
import { useEffect, useState } from "react";
import authgear, { IdentityType } from "@authgear/web";

const PROVIDER_ALIAS = "google";

function ProviderLinkButton() {
  const [linked, setLinked] = useState<boolean | null>(null);

  useEffect(() => {
    authgear.fetchUserInfo().then((userInfo) => {
      const isLinked = (userInfo.identities ?? []).some(
        (i) =>
          i.type === IdentityType.OAuth &&
          i.oauthProviderAlias === PROVIDER_ALIAS
      );
      setLinked(isLinked);
    });
  }, []);

  if (linked === null) {
    return <p>Loading…</p>;
  }

  return linked ? (
    <button onClick={startUnlink}>Unlink Google</button>
  ) : (
    <button onClick={startLink}>Link Google</button>
  );
}
```
{% endtab %}

{% tab title="JavaScript" %}
```html
<div id="provider-container"></div>

<script type="module">
  import authgear, { IdentityType } from "@authgear/web";

  const PROVIDER_ALIAS = "google";
  const container = document.getElementById("provider-container");

  const userInfo = await authgear.fetchUserInfo();
  const linked = (userInfo.identities ?? []).some(
    (i) =>
      i.type === IdentityType.OAuth &&
      i.oauthProviderAlias === PROVIDER_ALIAS
  );

  const button = document.createElement("button");
  if (linked) {
    button.textContent = "Unlink Google";
    button.onclick = startUnlink;
  } else {
    button.textContent = "Link Google";
    button.onclick = startLink;
  }
  container.appendChild(button);
</script>
```
{% endtab %}
{% endtabs %}

### Step 3: Link the provider

Call `startLinkOAuth()`. The SDK redirects the browser to Google's consent screen, where the user signs in and approves the link. After approval, Google redirects back to your `redirectURI`.

```typescript
function startLink() {
  authgear
    .startLinkOAuth({
      oauthProviderAlias: PROVIDER_ALIAS,
      redirectURI: "https://myapp.com/oauth-callback",
      state: "link_oauth",
    })
    .catch((err) => {
      console.error(err);
    });
}
```

| Option               | Type   | Required | Description                                                                                                       |
| -------------------- | ------ | -------- | ----------------------------------------------------------------------------------------------------------------- |
| `oauthProviderAlias` | string | Yes      | The provider alias configured in the Portal, e.g. `google`.                                                       |
| `redirectURI`        | string | Yes      | Where the browser returns after the flow. Must be registered in the Portal.                                       |
| `state`              | string | No       | An OAuth state value returned to your callback. You can use it to tell link, unlink, and sign-in callbacks apart. |

### Step 4: Unlink the provider

Call `startUnlinkOAuth()`. The SDK redirects the browser to an Authgear-hosted page with an unlink button. The user confirms there, and Authgear redirects back to your `redirectURI`.

```typescript
function startUnlink() {
  authgear
    .startUnlinkOAuth({
      oauthProviderAlias: PROVIDER_ALIAS,
      redirectURI: "https://myapp.com/oauth-callback",
      state: "unlink_oauth",
    })
    .catch((err) => {
      console.error(err);
    });
}
```

The options are the same as `startLinkOAuth()`. Use a different `state` value so your callback knows which action to finish.

### Step 5: Handle the callback

When the user returns to your `redirectURI`, finish the action. Read the `state` query parameter to choose between `finishLinkOAuth()` and `finishUnlinkOAuth()`, then send the user back to your account details page.

{% tabs %}
{% tab title="First Tab" %}
```tsx
import { useEffect } from "react";
import authgear from "@authgear/web";

function OAuthCallback() {
  useEffect(() => {
    async function finish() {
      // configure() must run before any finish method.
      await authgear.configure({
        clientID: "your-client-id",
        endpoint: "https://myapp.authgear.cloud",
        sessionType: "refresh_token",
      });

      const state = new URL(window.location.href).searchParams.get("state");

      try {
        if (state === "link_oauth") {
          await authgear.finishLinkOAuth();
        } else if (state === "unlink_oauth") {
          await authgear.finishUnlinkOAuth();
        }
        window.location.replace("/members");
      } catch (err) {
        console.error(err);
      }
    }
    finish();
  }, []);

  return <p>Finishing…</p>;
}
```
{% endtab %}

{% tab title="JavaScript" %}
```html
<script type="module">
  import authgear from "@authgear/web";

  // configure() must run before any finish method.
  await authgear.configure({
    clientID: "your-client-id",
    endpoint: "https://myapp.authgear.cloud",
    sessionType: "refresh_token",
  });

  const state = new URL(window.location.href).searchParams.get("state");

  try {
    if (state === "link_oauth") {
      await authgear.finishLinkOAuth();
    } else if (state === "unlink_oauth") {
      await authgear.finishUnlinkOAuth();
    }
    // Both actions are done. Return to the members portal.
    window.location.replace("/members");
  } catch (err) {
    console.error(err);
  }
</script>
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
`finishLinkOAuth()` and `finishUnlinkOAuth()` reject with an `OAuthError` if the user cancels or the flow fails. Catch the error and show the user a message before redirecting.
{% endhint %}

### Related pages

* [UserInfo endpoint reference](../../reference/apis/oauth-2.0-and-openid-connect-oidc/userinfo.md): the full `identities` claim.
