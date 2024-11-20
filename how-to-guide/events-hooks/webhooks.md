---
description: Webhooks is one of the supported hooks to receive events.
---

# Webhooks

To use webhooks you need to:

1. Deploy a webhook on your server.
2. Configure Authgear to deliver events to your webhook.

## Configure Authgear to deliver events to your webhook

{% tabs %}
{% tab title="Portal" %}
1. In the portal, go to **Advanced** > **Hooks**.
2. Add your webhooks in **Blocking Events** and **Non-Blocking Events**, depending on which event you want to listen to.
3. Click **Save**.
{% endtab %}

{% tab title="authgear.yaml" %}
```yaml
hook:
  blocking_handlers:
    - event: "user.pre_create"
      url: 'https://myapp.com/check_user_create'
  non_blocking_handlers:
    # listen to all events and filter events by type in request
    - events: ["*"]
      url: 'https://myapp.com/all_events'
    - events: ["user.created"]
      url: 'https://myapp.com/sync_user_creation'
```
{% endtab %}
{% endtabs %}

## Protocol

Events are delivered to your webhooks via **HTTPS**, so your server must support HTTPS.

Events are delivered to your webhooks with **POST** requests. You webhooks must return a HTTP status code within **2xx** range. Other status codes are considered as a failed delivery.

## Verifying signature

The request to your webhooks is signed with a secret key shared between Authgear and your hooks. You are **RECOMMENDED** to verify the signature and reject any requests with invalid signatures. This ensures the request originates from Authgear.

The signature is calculated as the hex encoded value of HMAC-SHA256 of the request body and included in the HTTP header `x-authgear-body-signature`.

To obtain the secret key, visit the portal and go to **Advanced** -> **Hooks** -> **Webhook Signature**. You may need to reauthenticate yourselves before you can reveal the secret key.

Here is the sample code of how to calculate the signature and verify it.

{% tabs %}
{% tab title="Go" %}
```go
package main

import (
    "crypto/hmac"
    "crypto/sha256"
    "crypto/subtle"
    "encoding/hex"
    "fmt"
    "io"
    "net/http"
)

// Obtain the secret in the portal.
const Secret = "SECRET"

// HMACSHA256String returns the hex-encoded string of HMAC-SHA256 code of body using secret as key.
func HMACSHA256String(data []byte, secret []byte) (sig string) {
    hasher := hmac.New(sha256.New, secret)
    _, _ = hasher.Write(data)
    signature := hasher.Sum(nil)
    sig = hex.EncodeToString(signature)
    return
}

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        b, err := io.ReadAll(r.Body)
        if err != nil {
            // Handle the error properly
            panic(err)
        }
        defer r.Body.Close()

        sigInHeader := []byte(r.Header.Get("X-Authgear-Body-Signature"))
        sig := []byte(HMACSHA256String(b, []byte(Secret)))

        // Prefer constant time comparison over == operator.
        if subtle.ConstantTimeCompare(sigInHeader, sig) != 1 {
            // The signature does not match
            // Do NOT trust the content of this webhook!!!
            panic(fmt.Errorf("%v != %v", string(sigInHeader), string(sig)))
        }

        // Continue your logic here.
    })
    http.ListenAndServe(":9999", nil)
}
```
{% endtab %}
{% endtabs %}
