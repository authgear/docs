---
description: >-
  Enable secure, automated authorization for your backend systems,
  microservices, and IoT devices, ensuring only trusted apps and devices can
  access your APIs.
---

# Machine-to-Machine (M2M) Applications

### What Are M2M Applications? <a href="#what-are-m2m-applications" id="what-are-m2m-applications"></a>

Machine-to-machine (M2M) applications enable backend services, CLIs, scheduled jobs, and smart devices to obtain access tokens and interact with APIs using dedicated credentials. Instead of human user authentication, these apps use their own credentials to securely access resources, streamlining automation and integration across your stack.

## Common Use Cases <a href="#common-use-cases" id="common-use-cases"></a>

* **Application Backends:** Facilitates secure transfer of data, files, or logs between microservices or external systems.
* **CLI Tools:** Allows tools running on developer or deployment machines to access APIs with short-lived tokens.
* **Scheduled Jobs & Daemons:** Empowers background tasks (e.g., cronjobs, queuing systems) to access protected resources safely.
* **IoT Devices:** Enables each smart device to authenticate independently and send data securely to cloud APIs.

## Set Up M2M Applications in Authgear <a href="#steps-to-set-up-m2m-applications-in-authgear" id="steps-to-set-up-m2m-applications-in-authgear"></a>

### 1. Register Your API Resources

Go to the **API Resources** page in the Authgear portal:

* Create the API resource your apps/devices will access.
* When registering the resource, specify its **identifier**—this acts as the unique URI representing your API (for example, `https://myapi.com/api`).
  * This identifier will be set as the `aud` (audience) claim in the JWT access tokens issued for this resource.
* Define **scopes** for granular permissions, such as `read:data`, `write:data`, or `manage:config`.
* Scopes specify exactly **what** operations clients can perform on your API.

### 2. Register Your Application

In the **Applications** page:

* Create a client application for each backend service, CLI tool, job, or device that needs API access.
* Assign the relevant **API resources** and **scopes** to the application, ensuring least-privilege access.
* After creating the application, you’ll be able to view its **Client ID** and **Client Secret** in the portal. Keep the Client Secret secure. It should never be exposed in client-side code or public repositories.

### 3. Request a Token Using the Client Credentials Flow

The **Client Credentials Flow** defined in [OAuth 2.0 RFC 6749, section 4.4](https://tools.ietf.org/html/rfc6749#section-4.4) is designed for non-user, automated access.

To obtain an access token in your backend, use the Client ID and Client Secret from your registered application in the portal:

{% tabs %}
{% tab title="curl" %}
```
curl --request POST \
  --url https://myproject.authgear.cloud/oauth2/token \
  --header 'Content-Type: application/x-www-form-urlencoded' \
  --data grant_type=client_credentials \
  --data resource=IDENTIFIER_OF_API_RESOURCE \
  --data client_id=CLIENT_ID \
  --data client_secret=CLIENT_SECRET
```
{% endtab %}

{% tab title="Python" %}
```python
import urllib.parse
import urllib.request
import json

url = "https://myproject.authgear.cloud/oauth2/token"
headers = {
    "Content-Type": "application/x-www-form-urlencoded"
}
data = {
    "grant_type": "client_credentials",
    "resource": "IDENTIFIER_OF_API_RESOURCE",
    "client_id": "CLIENT_ID",
    "client_secret": "CLIENT_SECRET"
}

encoded_data = urllib.parse.urlencode(data).encode('utf-8')
req = urllib.request.Request(url, data=encoded_data, headers=headers, method='POST')

with urllib.request.urlopen(req) as response:
    response_status = response.getcode()
    response_body = response.read().decode('utf-8')
    print("Response Status Code:", response_status)
    print("Response Body:", json.loads(response_body))

```
{% endtab %}

{% tab title="Go" %}
```go
package main

import (
  "fmt"
  "io/ioutil"
  "net/http"
  "net/url"
  "strings"
)

func main() {
  data := url.Values{}
  data.Set("grant_type", "client_credentials")
  data.Set("resource", "IDENTIFIER_OF_API_RESOURCE")
  data.Set("client_id", "CLIENT_ID")
  data.Set("client_secret", "CLIENT_SECRET")

  req, _ := http.NewRequest("POST", "https://myproject.authgear.cloud/oauth2/token", strings.NewReader(data.Encode()))
  req.Header.Add("Content-Type", "application/x-www-form-urlencoded")

  resp, _ := http.DefaultClient.Do(req)
  defer resp.Body.Close()

  body, _ := ioutil.ReadAll(resp.Body)

  fmt.Println("Response Status Code:", resp.StatusCode)
  fmt.Println("Response Body:", string(body))
}

```
{% endtab %}

{% tab title="Node JS" %}
```javascript
async function makeRequest() {
  const url = "https://myproject.authgear.cloud/oauth2/token";
  const data = new URLSearchParams();
  data.append("grant_type", "client_credentials");
  data.append("resource", "IDENTIFIER_OF_API_RESOURCE");
  data.append("client_id", "CLIENT_ID");
  data.append("client_secret", "CLIENT_SECRET");

  const response = await fetch(url, {
    method: "POST",
    headers: {
      "Content-Type": "application/x-www-form-urlencoded",
    },
    body: data,
  });

  const responseBody = await response.json();
  console.log("Response Status Code:", response.status);
  console.log("Response Body:", responseBody);
}

makeRequest();

```
{% endtab %}
{% endtabs %}

You’ll receive an access token that includes only the permitted scopes for the client.

For details on token structure and claims, see [m2m-tokens.md](../reference/tokens/m2m-tokens.md "mention").

### 4. Access Protected APIs

Send requests to your API by including the access token in the `Authorization` header:

```
Authorization: Bearer <access_token>
```

Your backend can now verify the token and authorize the permitted operations.

### 5. Verify Tokens on the API Server

Always validate incoming JWT tokens in your API servers:

* Verify issuer, audience, scopes, expiration, and signature.
* Reject requests with invalid or expired tokens.

Find sample code and best practices at [jwt.md](backend-api/jwt.md "mention").

