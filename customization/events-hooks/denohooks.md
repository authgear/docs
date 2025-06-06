---
description: JavaScript / TypeScript Hooks is one of the supported hooks to receive events.
---

# JavaScript / TypeScript Hooks

JavaScript / TypeScript Hooks are written as a [ES2015 module](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules). The module is executed by [Deno](https://deno.land/).

The module **MUST** have a [default export](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export#description) of a function taking 1 argument. The argument is the [event](./#event-shape). The function can either be synchronous or asynchronous.

If the Hook is registered for a blocking event, the function **MUST** return a value according to the [specification](./#blocking-events).

The Hooks **DO NOT** have access to file, or environment. They only have access to external network.

The stdout and the stderr of the Hooks are both ignored. Your hooks **MUST NOT** assume anything on the arguments and the stdin of the module.

## Configure Authgear to deliver events to your Hooks

{% tabs %}
{% tab title="Portal" %}
1. In the portal, go to **Advanced** > **Hooks**.
2. Add your Hooks in **Blocking Events** and **Non-Blocking Events**, depending on which event you want to listen to.
3. Click **Save**.
{% endtab %}
{% endtabs %}

## Examples

Here is an example of a Hook for a blocking event.

```typescript
import { HookEvent, HookResponse } from "https://deno.land/x/authgear_deno_hook@v1.0.0/mod.ts";
export default async function(e: HookEvent): Promise<HookResponse> {
  // This hook simply allows the operation, which is identical to no-op.
  return { is_allowed: true };
}
```

An example to mutate a JWT token

```typescript
import {HookResponse, EventOIDCJWTPreCreate } from "https://deno.land/x/authgear_deno_hook@v1.0.0/mod.ts";

export default async function(event: EventOIDCJWTPreCreate): Promise<HookResponse> {
  return { 
    is_allowed: true,
    mutations: {
      jwt:{
        payload:{
          ...event.payload.jwt.payload, //the original payload in the jwt
          "https://myapp.com": {
            "custom_field": "custom_value"
          }
        }
      }
    }
  };
}
```

Here is an example of a Hook for a non-blocking event.

```typescript
import { HookEvent } from "https://deno.land/x/authgear_deno_hook@v0.3.0/mod.ts";
export default async function(e: HookEvent): Promise<void> {
  // This hook does nothing, which is identical to no-op.
}
```

## TypeScript Definition

[https://deno.land/x/authgear\_deno\_hook](https://deno.land/x/authgear\_deno\_hook) is a TypeScript definition that aids you in writing a Hook. You can see the full definition at [https://deno.land/x/authgear\_deno\_hook/mod.ts](https://deno.land/x/authgear\_deno\_hook/mod.ts)

If you are a Visual Studio Code user, you can [set up your editor](https://deno.land/manual@v1.27.2/references/vscode\_deno) to take full advantage of the definition.

Alternatively, you can edit your hook and use the Deno CLI to typecheck.

```bash
$ deno check YOUR_HOOK.ts
```
