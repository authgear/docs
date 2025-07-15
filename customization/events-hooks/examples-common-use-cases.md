---
description: >-
  Common usage of events and hooks to modify the behavior of an authentication
  on Authgear
---

# Examples: Common Use Cases

Here are some example of using Hook to add actions to Authgear during the authentication flow. This allows extension of security features such as more granular access control, adaptive MFA and variable rate limit.

Navigate to **Advanced** > **Hooks** in the Authgear portal. Next, click on **Add** under the **Blocking Events** section. Select "TypeScript" and then "Edit Script" to write code that will be executed on your desired events.&#x20;

Webhook can also be used if you want to run the code in your server. See [webhooks.md](webhooks.md "mention") for more details.

## Allow signups only from inside the corporate network

You can use the **user.pre\_create** event to allow signups only from a certain IP.

For example, only allow signup when user is in the network `123.45.67.89` :

```typescript
export default async function (
  e: EventUserPreCreate
): Promise<HookResponse> {
  const allowedIp = "123.45.67.89"; // The IP address of the corporate network
  if (e.context.ip_address == allowedIp) {
    return {
      is_allowed: true,
    };
  } else {
    return {
      is_allowed: false,
      reason: "You are not allow to sign up to this organisation",
      title: "Sign-up not allowed!",
    };
  }
}
```

## Allowing access for specific geo location

You can use the **authentication.post\_identified** event to block or allow users from a certain location.

For example, only allow signups and logins from Hong Kong:

```typescript
export default async function (
  e: EventAuthenticationPostIdentified
): Promise<EventAuthenticationPostIdentifiedHookResponse> {
  if (e.context.geo_location_code === "HK") {
    return {
      is_allowed: true,
    };
  } else {
    return {
      is_allowed: false,
    };
  }
}
```

## Allow access according to user roles for a certain application

You can use **authentication.post\_identified** to block or allow user authenticate in a certain app according to their roles, standard or custom attributes.

For example, if you want to allow only users with role `sales` to access the app `crm-system` with client id is `c8da9b322e1f494e`:

```typescript
export default async function (
  e: EventAuthenticationPostIdentified
): Promise<EventAuthenticationPostIdentifiedHookResponse> {
  if (
    e.context.client_id === "c8da9b322e1f494e" &&
    e.payload.authentication_context?.authentication_flow?.type != null &&
    ["login", "signup"].includes(e.payload.authentication_context?.authentication_flow?.type)
  ) {
    const user = e.payload.authentication_context.user
    if (user?.roles?.includes("sales")) {
      return {
        is_allowed: true,
      };
    } else if (user?.custom_attributes?.can_access_crm === "true") {
      // Alternatively, use custom_attributes to determine if the user is allowed to access the app
      return {
        is_allowed: true,
      };
    } else {
      return {
        is_allowed: false,
        reason: "You don't have permission to use this app"
      };
    }
  }
  // Allow login or signups of other clients
  return {
    is_allowed: true,
  };
}
```

## Allow access according to email domain

You can use **authentication.post\_identified** to block user from signing up in your system if they are not using a specific email domain.

For example, you only want user with email domain `@authgear.com` to be able to signup:

```typescript
export default async function (
  e: EventAuthenticationPostIdentified
): Promise<EventAuthenticationPostIdentifiedHookResponse> {
  const email = e.payload.identification.identity?.claims?.email
  if (typeof email === "string" && email.endsWith("@authgear.com")) {
    return {
      is_allowed: true,
    };
  }
  // Block signup of all other emails
  return {
    is_allowed: false,
  };
}
```

## Block login during weekends

You can use **authentication.post\_identified** to block user from logging in during weekends.

For example, if your business only operate during weekdays, therefore you do not want any user login during weekends:

```typescript
export default async function (
  e: EventAuthenticationPostIdentified
): Promise<EventAuthenticationPostIdentifiedHookResponse> {
  const today = new Date();
  // 0 is sunday, and 6 is saturday
  if (today.getDay() === 0 || today.getDay() === 6) {
    return {
      is_allowed: false,
    };
  }
  return {
    is_allowed: true,
  };
}
```

Note: Even login is blocked during weekends, refresh token and access tokens issued during weekdays will not be invalidated.

## Block user login according to authentication method

You can use **authentication.pre\_authenticated** to block user login according to the [amr.md](amr.md "mention") used during authentication.

For example, if you want to block users who **DID NOT** use `mfa` (multi-factor authentication) during the authentication:

```typescript
export default async function (
  e: EventAuthenticationPreAuthenticated
): Promise<EventAuthenticationPreAuthenticatedHookResponse> {
  if (!e.payload.authentication_context.amr?.includes("mfa")) {
    return {
      is_allowed: false,
    };
  }
  return {
    is_allowed: true,
  };
}
```

## Enable bot protection under specific conditions

Use **authentication.pre\_initialize** to enable bot protection under specific conditions.

For example, if you want to display captcha only for user outside Hong Kong:

Firstly, enable [bot-protection.md](../../security/bot-protection.md "mention") in the portal and set the requirements to "Never"&#x20;

Then, return `bot_protection.mode` in your `authentication.pre_initialize` hook:

```typescript
export default async function (
  e: EventAuthenticationPreInitialize
): Promise<EventAuthenticationPreInitializeHookResponse> {
  if (e.context.geo_location_code !== "HK") {
    return {
      is_allowed: true,
      bot_protection: {
        mode: "always",
      },
    };
  }
  // Else, simply allow the login
  return {
    is_allowed: true,
  };
}
```

This overrides the original `mode` of bot\_protection in your config. Therefore, bot\_protection will be turned on if the user is trying to signup or login outside Hong Kong.

## Adaptive MFA: Require MFA only for users with high risk

You can use **authentication.pre\_authenticated** to implement Adaptive MFA.

For example, you consider logins from outside `HK` is at a higher risk, therefore MFA should be required:

```typescript
export default async function (
  e: EventAuthenticationPreAuthenticated
): Promise<EventAuthenticationPreAuthenticatedHookResponse> {
  if (e.context.geo_location_code !== "HK") {
    return {
      // Allow the login with a mfa contraint
      is_allowed: true,
      constraints: {
        amr: ["mfa"],
      },
    };
  }
  // Else, simply allow the login
  return {
    is_allowed: true,
  };
}
```

If `constraints.amr` with value `["mfa"]` is returned in the response, depending on the authentication flow type:

* Signup / Promote: If the user does not have any secondary authenticator setup during the flow, a step will be added at the end of the flow to force user to setup a secondary authenticator.&#x20;
* Login / Re-authentication: If the user does not use any secondary authenticator during the flow, a step will be added at the end of the flow for 2FA. If the user never setup 2FA before, the flow fail.
* Account Recovery: No effect, because account recovery does not support 2FA.

Learn more about AMR in [amr.md](amr.md "mention").

## Advanced: Applying stricter rate limits for authentications for a certain IP range

You can apply a stricter rate limit in an authentication for a certain IP range flow using hooks.

For example, you want to limit account enumeration to **5 per minute if the request origins from a data center IP address**, and **10 attempts per minute in any other requests**.

### Adjust base rate limit

Firstly, add this rate limit configuration in the project config. (Portal > Advanced > Edit Config)

```yaml
authentication:
  rate_limits:
    account_enumeration:
      per_ip:
        enabled: true
        period: 1m
        burst: 10
```

This sets the base rate limit of account enumeration to 10/minute.

### Override rate limit in the hook by a weight

Then, create the following hook in the **authentication.pre\_initialize** event:

```typescript
function ipToBinary(ip: string): string {
  return ip
    .split(".")
    .map((octet) => parseInt(octet, 10).toString(2).padStart(8, "0"))
    .join("");
}

function cidrToPrefix(cidr: string): string {
  const [ip, lengthStr] = cidr.split("/");
  const prefixLength = parseInt(lengthStr, 10);
  const ipBinary = ipToBinary(ip);
  return ipBinary.substring(0, prefixLength);
}

export default async function (
  e: EventAuthenticationPreInitialize
): Promise<EventAuthenticationPreInitializeHookResponse> {
  if (e.context.ip_address == null) {
    // ip unknown, block it.
    return {
      is_allowed: false,
    };
  }

  const ipBinary = ipToBinary(e.context.ip_address);

  // ip ranges from https://www.gstatic.com/ipranges/cloud.json
  const dataCenterIPRanges = ["34.125.0.0/16", "34.124.24.0/21"];
  // 34.125.0.0/16
  if (dataCenterIPRanges.some((cidr) => ipBinary.startsWith(cidrToPrefix(cidr)))) {
    return {
      is_allowed: true,
      rate_limits: {
        "authentication.account_enumeration": {
          weight: 2,
        },
      },
    };
  } else {
    return {
      is_allowed: true,
    };
  }
}
```

By setting `"rate_limits.authentication.account_enumeration.weight"` to 2, any attempt of account enumeration will contribute `2` attempts to the rate limit. Therefore, only 5 attempts are  allowed in 1 minute. (10 / 2 = 5). Learn more in [#override-rate-limits](blocking-events.md#override-rate-limits "mention")
