# Examples: Common Use Cases

## Allow signups only from inside the corporate network

You can use the **user.pre\_create** event to allow signups only from a certain IP.

For example, only allow signup when user is in the network `123.45.67.89` :

```typescript
export default async function (e: EventUserPreCreate): Promise<HookResponse> {
  // Write your hook with the help of the type definition.
  const allowedIp = "123.45.67.89";
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
