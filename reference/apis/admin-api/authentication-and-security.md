---
description: >-
  The Admin API is protected by cryptographic keys. Learn how to generate a
  valid JWT to authorize your request in this article.
---

# Authentication and Security

## Accessing the Admin API GraphQL endpoint

Accessing the Admin API GraphQL endpoint requires your server to generate a valid JWT and include it as `Authorization` HTTP header.

## Obtaining the private key for signing JWT

* Go to **Advanced** -> **Admin API**
* Click **Download** to download the private key. Make it available to your server.
* **Copy** the key ID. Make it available to your server.

## Generating the JWT with the private key

#### Sample code to generate the JWT

Here is the sample code of how to generate the JWT with the private key.

{% tabs %}
{% tab title="Go" %}
```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
	"time"

	"github.com/lestrrat-go/jwx/v2/jwa"
	"github.com/lestrrat-go/jwx/v2/jwk"
	"github.com/lestrrat-go/jwx/v2/jws"
	"github.com/lestrrat-go/jwx/v2/jwt"
)

// Replace "myapp" with your project ID here.
// It is the first part of your Authgear endpoint.
// e.g. The project ID is "myapp" for "https://myapp.authgear.cloud"
const ProjectID = "myapp"

// Replace "mykid" with the key ID you see in the portal.
const KeyID = "mykid"

func main() {
	// Replace the following call with your own way to get the private key.
	f, err := os.Open("private-key.pem")
	if err != nil {
		panic(err)
	}
	defer f.Close()
	jwkSet, err := jwk.ParseReader(f, jwk.WithPEM(true))
	if err != nil {
		panic(err)
	}
	key, _ := jwkSet.Key(0)
	key.Set("kid", KeyID)

	now := time.Now().UTC()
	payload := jwt.New()
	_ = payload.Set(jwt.AudienceKey, ProjectID)
	_ = payload.Set(jwt.IssuedAtKey, now.Unix())
	_ = payload.Set(jwt.ExpirationKey, now.Add(5*time.Minute).Unix())

	// The alg MUST be RS256.
	alg := jwa.RS256
	hdr := jws.NewHeaders()
	hdr.Set("typ", "JWT")

	buf, err := json.Marshal(payload)
	if err != nil {
		panic(err)
	}

	token, err := jws.Sign(buf, jws.WithKey(alg, key, jws.WithProtectedHeaders(hdr)))
	if err != nil {
		panic(err)
	}

	fmt.Printf("%v\n", string(token))
}
```
{% endtab %}

{% tab title="Python" %}
```python
import jwt
from datetime import datetime, timedelta

private_key = open("private-key.pem", "r").read()

# Replace "myapp" with your project ID here. 
# It is the first part of your Authgear endpoint. 
# e.g. The project ID is "myapp" for "https://myapp.authgear.cloud"
PROJECT_ID = "myapp"

# Replace "mykid" with the key ID you see in the portal.
KEY_ID = "mykid"

now = datetime.now()

payload = {
    "aud": [PROJECT_ID],
    "iat": int(now.timestamp()),
    "exp": int((now + timedelta(minutes=5)).timestamp()),
}

token = jwt.encode(
    payload,
    private_key,
    "RS256",
    headers={
        "kid": KEY_ID,
    },
)

print(token)
```
{% endtab %}

{% tab title="Node.js" %}
This example uses Express.js and the JsonWebToken package. So, first install both packages using the following commands:

```sh
npm install express
```

and

```sh
npm install jsonwebtoken
```

Here's the code for the Express.js app that generates the JWT:

```javascript
const express = require("express");
const node_jwt = require('jsonwebtoken');
const fs = require('fs');
const app = express();
const port = 3002;

app.get('/', (request, response) => {
    // Your project ID is the first part of your Authgear endpoint. 
    // e.g. The project ID is "myapp" for "https://myapp.authgear.cloud"
    // You can find your key ID in the Portal.
    
    const project_id = ""; //Place your Authgear project id
    const key_id = ""; // Place your Authgear key ID
    const expiresAt = Math.floor(Date.now() / 1000) + (60 * 5); //the current value means token will expire in 5 minutes.
    
    //Payload to include in JWT
    const claims = {
        aud: project_id,
        iat: Math.floor(Date.now() / 1000),
        exp: expiresAt

    }

    const privateKey = fs.readFileSync("key.pem"); //Read value from the downloaded key file
    const header = {"typ": "JWT", "kid": key_id, "alg": "RS256"}
    const jwt = node_jwt.sign(claims, privateKey, { header: header });

    response.send("Generated JWT: " + jwt);
});

app.listen(port, () => {
    console.log("server started on port " + port);
});
```
{% endtab %}

{% tab title="PHP" %}
First, install the Firebase PHP JWT package using this command:

```sh
composer require firebase/php-jwt
```

The PHP code for generating JWT:

```php
<?php
require 'vendor/autoload.php';
use Firebase\JWT\JWT;

// Your project ID is the first part of your Authgear endpoint. 
// e.g. The project ID is "myapp" for "https://myapp.authgear.cloud"
// You can find your key ID in the Portal.

$project_id = ""; // Place your Authgear project id
$key_id = ""; // Place your Authgear key ID
$expiresAt = time() + (60*5); //the current value means token will expire in 5 minutes.

//Payload to include in JWT
$claims = [
    "aud" => $project_id,
    "iat" => time(),
    "exp" => $expiresAt
];

$privateKey = file_get_contents("./key.pem"); //Read value from the downloaded key file
$header = [
    "typ" => "JWT",
        "kid" => $key_id,
        "alg" => "RS256"
    ];
$jwt = JWT::encode($claims, $privateKey, "RS256", null, $header);

echo $jwt;
```
{% endtab %}
{% endtabs %}

#### Example of the JWT header

```json
{
  "alg": "RS256",
  "kid": "REPLACE_YOUR_KEY_ID_HERE",
  "typ": "JWT"
}
```

#### Example of the JWT payload

```json
{
  "aud": [
    "REPLACE_YOUR_PROJECT_ID_HERE"
  ],
  "exp": 1136257445,
  "iat": 1136171045
}
```

## Including the JWT in the HTTP request

After generating the JWT, you must include it in **EVERY** request you send to the Admin API endpoint. Here is how it looks like

```
Authorization: Bearer <JWT>
```

The header is the standard Authorization HTTP header. The token type **MUST** be Bearer.

## Optional: Caching the JWT

As you can see in the sample code, you expiration time of the JWT is 5 minutes. You make it last longer and cache it to avoid generating it on every request.

## Admin API Key rotation

You should regularly change the API key used to authenticate API requests. It enhances security by minimizing the impact of compromised keys.

To rotate the API key

1. Go to **Portal** > **Advanced** > **Admin API**
2. Under "List of Admin API keys", click "Generate new key pair"
3. At this point both keys can be used to authenticate the admin API requests.
4. Make sure all your systems is updated to use the new key
5. Delete the old API key
