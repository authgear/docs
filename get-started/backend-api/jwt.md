---
description: >-
  Authenticate the incoming HTTP requests by validating JWT in your application
  server
---

# Validate JWT in your backend

[![LLM | View as markdown](https://img.shields.io/badge/LLM-View%20as%20markdown-blue)](https://raw.githubusercontent.com/authgear/docs/refs/heads/main/get-started/backend-api/jwt.md)

In this section, we will go through how to decode the JWT token to obtain the currently logged-in user.

Before we start, make sure the option **Issue JWT as access token** is enabled in your Application settings in the Portal.

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt="" width="375"><figcaption><p>Enable this option in application settings in the portal</p></figcaption></figure>

With the **Issue JWT as access token** option turned on in your application, Authgear will issue JWT as access tokens. The incoming HTTP requests should include the access token in their `Authorization` headers. Without setting the reverse proxy, your backend server can use your Authgear **JWKS** to verify the request and decode user information from the JWT access token.

## Find the JSON Web Key Sets (JWKS) endpoint

This Discovery endpoint serves as a JSON document containing the OpenID Connect configuration of your app. It includes the authorization endpoint, the token endpoint, and the JWKS endpoint.

`https://<YOUR_AUTHGEAR_ENDPOINT>/.well-known/openid-configuration`

The JSON Web Key Sets (JWKS) endpoint can be found in `jwks_uri` in the configuration.

**OpenID Connect Configuration JSON Example**

Here is [an example of how it looks](https://accounts.portal.authgear.com/.well-known/openid-configuration).

```json
{
    "issuer": "https://project-id.authgear.cloud",
    "authorization_endpoint": "https://project-id.authgear.cloud/oauth2/authorize",
    "jwks_uri": "https://project-id.authgear.cloud/oauth2/jwks", // the JWKS endpoint
    ...
}
```

## Decode user from an access token

Follow this step-by-step example to verify and decode the JWT token.

{% tabs %}
{% tab title="Python" %}
**Step 1: Install packages**

```bash
pip install cryptography
pip install PyJWT
```

**Step 2: Find the JSON Web Key Sets (JWKS) endpoint**

Define a function to find the JWKS endpoint from the OpenID Connect configuration. Use your Authgear endpoint as the `base_address`

```python
import json
from contextlib import closing
from urllib.request import urlopen

base_address = "https://<your_app_endpoint>"

def fetch_jwks_uri(base_address):
    doc_url = base_address + "/.well-known/openid-configuration"
    with closing(urlopen(doc_url)) as f:
        doc = json.load(f)
    jwks_uri = doc["jwks_uri"]
    if not jwks_uri:
        raise Exception('Failed to fetch jwks uri.')
    return jwks_uri
```

**Step 3: Get the JWT token from the Authorization header**

Define a function to extract the access token from the Authorization header in the incoming request. It should look like `Authorization: Bearer <access_token>`.

```python
def parse_header(authz_header):
    parts = authz_header.split(" ")
    if len(parts) != 2:
        return

    scheme = parts[0]
    if scheme.lower() != "bearer":
        return

    return parts[1]
```

**Step 4: Verify and decode the JWT token**

Here we show an example of using the Flask web framework to guard a path. You may need to adjust some of the codes to suit your technologies.

```python
from flask import request
import jwt
from jwt import PyJWKClient

@app.route("/hello")
def hello():
    authz_header = request.headers.get("Authorization")
    if not authz_header:
        return {
            "message": "authz header not found"
        }

    # get jwt token from Authorization header
    token = parse_header(authz_header)
    if token:
        try:
            # fetch jwks_uri from the Authgear Discovery Endpoint
            jwks_uri = fetch_jwks_uri(base_address)
            # Reuse PyJWKClient for better performance
            jwks_client = PyJWKClient(jwks_uri)
            signing_key = jwks_client.get_signing_key_from_jwt(token)
            user_data = jwt.decode(
                token,
                signing_key.key,
                algorithms=["RS256"],
                audience=base_address,
                options={"verify_exp": True},
            )
            return {
                "message": "Hello!",
                "user_data": user_data
            }
        except:
            return {
                "message": "JWT decode failed"
            }
    else:
        return {
            "message": "no token"
        }
```
{% endtab %}

{% tab title="Node.js" %}
**Step 1: Install dependencies**

```bash
npm install --save axios jwks-rsa jsonwebtoken
```

**Step 2: Find the JWKS Endpoint**

Use the following method to get the JWKS URI (you'll need to URI to extract the public signing key from a JWT).

```javascript
const appUrl = ""; //place your authgear app endpoint here
const getJwksUri = async (appUrl) => {
    const config_endpoint = appUrl + "/.well-known/openid-configuration";
    const data = await axios.get(config_endpoint);
    return data.data.jwks_uri;
}
```

**Step 3: Extract JWT from Request Header**

Use the following code to extract only the token part from a `Bearer [token]` authorization header in your Express app:

```javascript
const express = require("express");
const axios = require("axios");
const node_jwt = require('jsonwebtoken');
const jwksClient = require('jwks-rsa');

const app = express();
const port = 3002;
app.get('/', async (req, res) => {

    const requestHeader = req.headers;
    if (requestHeader.authorization == undefined) {
        res.send("Invalid header");
        return;
    }
    const authorizationHeader = requestHeader.authorization.split(" ");
    const access_token = authorizationHeader[1];

}
```

**Step 4: Decode Access Token**

Next, decode the access token so that you can extract the JWT `kid` from the result. You'll need this \`kid to get the public signing key. Use the following code to decode the JWT:

```javascript
const decoded_access_token = node_jwt.decode(access_token, {complete: true});
```

**Step 5: Get JWT Signing Keys and Verify the JWT**

Use the following code to extract the JWT public keys then verify the JWT using the keys:

```javascript
const jwks_uri = await getJwksUri(appUrl);
    const client = jwksClient({
        strictSsl: true,
        jwksUri: jwks_uri
    });
    const signing_key = await client.getSigningKey(decoded_access_token.header.kid);

    try {
        const verify = node_jwt.verify(access_token, signing_key.publicKey, { algorithms: ['RS256'] });
        res.send(JSON.stringify(verify))
    }
    catch(error) {
        res.send(error);  
    }
    
```

Here's what your Express app should look like after putting the code in all the steps together:

```javascript
const express = require("express");
const axios = require("axios");
const node_jwt = require('jsonwebtoken');
const jwksClient = require('jwks-rsa');

const app = express();
const port = 3002;

const appUrl = "https://demo-1-ea.authgear.cloud";
const getJwksUri = async (appUrl) => {
    const config_endpoint = appUrl + "/.well-known/openid-configuration";
    const data = await axios.get(config_endpoint);
    return data.data.jwks_uri;
}

app.get('/', async (req, res) => {

    const requestHeader = req.headers;
    if (requestHeader.authorization == undefined) {
        res.send("Invalid header");
        return;
    }
    const authorizationHeader = requestHeader.authorization.split(" ");
    const access_token = authorizationHeader[1];
    const decoded_access_token = node_jwt.decode(access_token, {complete: true});
    const jwks_uri = await getJwksUri(appUrl);
    const client = jwksClient({
        strictSsl: true,
        jwksUri: jwks_uri
    });
    const signing_key = await client.getSigningKey(decoded_access_token.header.kid);

    try {
        const verify = node_jwt.verify(access_token, signing_key.publicKey, { algorithms: ['RS256'] });
        res.send(JSON.stringify(verify))
    }
    catch(error) {
        res.send(error);  
    }
});

app.listen(port, () => {
    console.log(`server started on port ${port}`);
});
```
{% endtab %}

{% tab title="Go" %}
Use your Authgear endpoint as `base_address`

```go
import (
    "context"
    "encoding/json"
    "fmt"
    "net/http"
    "regexp"
    "time"

    "github.com/lestrrat-go/jwx/v3/jwk"
    "github.com/lestrrat-go/jwx/v3/jwt"
)


var (
    authzHeaderRegexp = regexp.MustCompile("(?i)^Bearer (.*)$")
    baseAddress       = "https://<your_app_endpoint>"
)

type OIDCDiscoveryDocument struct {
    JWKSURI string `json:"jwks_uri"`
}

func FetchOIDCDiscoveryDocument(endpoint string) (*OIDCDiscoveryDocument, error) {
    resp, err := http.DefaultClient.Get(endpoint)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()

    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf(
            "failed to fetch discovery document: unexpected status code: %d",
            resp.StatusCode,
        )
    }

    var document OIDCDiscoveryDocument
    err = json.NewDecoder(resp.Body).Decode(&document)
    if err != nil {
        return nil, err
    }
    return &document, nil
}

func FetchJWK(baseAddress string) (jwk.Set, error) {
    doc, err := FetchOIDCDiscoveryDocument(
        baseAddress + "/.well-known/openid-configuration",
    )
    if err != nil {
        return nil, err
    }

    set, err := jwk.Fetch(context.Background(), doc.JWKSURI)
    return set, err
}

// DecodeUser parse request Authorization header and obtain user id and claims
func DecodeUser(r *http.Request) (string, bool, bool, error)
    // fetch jwks_uri from Authgear
    // you can cache the value of jwks using jwk.Cache to have better performance
    set, err := FetchJWK(baseAddress)
    if err != nil {
        return "", false, false, fmt.Errorf("failed to fetch JWK: %s", err)
    }

    // parse jwt token
    token, err := jwt.ParseRequest(
        r, 
        // This may not work out of the box depending on the jwk.Set.
        // Please read about requirements for "kid" and "alg" (and possibly
        // "WithDefaultKey") when using jwk.Set in the jwt.WithKeySet documentation.
        jwt.WithKeySet(set),
    )
    if err != nil {
        return "", false, false, fmt.Errorf("invalid token: %s", err)
    }

    // validate jwt token
    err = jwt.Validate(token,
        jwt.WithClock(jwt.ClockFunc(
            func() time.Time { return time.Now().UTC() },
        )),
        jwt.WithAudience(baseAddress),
    )
    if err != nil {
        return "", false, false, fmt.Errorf("invalid token: %s", err)
    }

    var verified bool
    var anonymous bool
    // ignore errors -- if the the claim does not exist, it doesn't matter
    _ = token.Get("https://authgear.com/claims/user/is_verified", &verified)
    _ = token.Get("https://authgear.com/claims/user/is_anonymous", &anonymous)

    return token.Subject(), verified, anonymous, nil
}

func handler(w http.ResponseWriter, r *http.Request) {
    // decode user example
    userid, isUserVerified, isAnonymousUser, err := DecodeUser(r)

    // ... your handler logic
}
```
{% endtab %}

{% tab title="Java" %}
The following example uses Spring Boot.

**Step 1: Install dependencies**

Add the following dependencies to your build.gradle file:

```gradle
dependencies {
	implementation("com.nimbusds:nimbus-jose-jwt:10.2")
	implementation("org.json:json:20250107")
}
```

Then add the following imports to the top of your controller file:

```java
import com.nimbusds.jose.jwk.JWKSet;
import com.nimbusds.jose.jwk.RSAKey;
import com.nimbusds.jwt.JWTClaimsSet;
import com.nimbusds.jwt.SignedJWT;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.List;
```

**Step 2: Get JWKS Endpoint**

Implement the following method to fetch the JWKS URI:

```java

private static String fetchJwksUri(String baseAddress) throws Exception {
	String docUrl = baseAddress + "/.well-known/openid-configuration";
	HttpURLConnection conn = (HttpURLConnection) new URL(docUrl).openConnection();
	conn.setRequestMethod("GET");

	try (BufferedReader reader = new BufferedReader(new InputStreamReader(conn.getInputStream()))) {
		StringBuilder response = new StringBuilder();
		String line;
		while ((line = reader.readLine()) != null) {
			response.append(line);
		}

		String jwksUri = new org.json.JSONObject(response.toString()).getString("jwks_uri");
		if (jwksUri == null || jwksUri.isEmpty()) {
			throw new Exception("Failed to fetch JWKS URI.");
		}
		return jwksUri;
	}
}
```

**Step 3: Get Signing Key**

Get the signing key from the JWK using the following method:

```java
private static RSAKey getSigningKeyFromJwks(String jwksUri, String token) throws Exception {
	JWKSet jwkSet = JWKSet.load(new URL(jwksUri));
	List<com.nimbusds.jose.jwk.JWK> keys = jwkSet.getKeys();

	SignedJWT signedJWT = SignedJWT.parse(token);
	String keyId = signedJWT.getHeader().getKeyID();

	return keys.stream()
			.filter(jwk -> jwk.getKeyID().equals(keyId))
			.findFirst()
			.map(jwk -> (RSAKey) jwk)
			.orElse(null);
}
```

**Step 4: Validate JWT**

To demonstrate how to validate a JWT, we'll implement a `validateJWT` endpoint in a Spring Boot application. The endpoint will read access tokens from the bearer authorization header.

It will call the `fetchJwksUri()` and `getSigningKeyFromJwks()` from steps 1 and 2 to get the JWK URI and signing key required to parse the JWT.

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.List;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RestController;

import com.nimbusds.jose.jwk.JWKSet;
import com.nimbusds.jose.jwk.RSAKey;
import com.nimbusds.jwt.JWTClaimsSet;
import com.nimbusds.jwt.SignedJWT;


@SpringBootApplication
@RestController
public class DemoApplication {

	//paste implementation for fetchJwksUri() method below this line.
	
	
	//paste implemetation of getSigningKeyFromJwks() method below this line.

	private static final String BASE_ADDRESS = ""; //place your authgear app endpoint here

	@GetMapping("/validateJwt")
	public Object validateJwt(@RequestHeader("Authorization") String authorizationHeader) {
		if (authorizationHeader == null || !authorizationHeader.startsWith("Bearer ")) {
			return new ResponseMessage("authorization header not found");
		}

		String token = authorizationHeader.substring(7); // Extract token

		try {
			// Fetch JWKS URI dynamically
			String jwksUri = fetchJwksUri(BASE_ADDRESS);

			// Get signing key from JWKS
			RSAKey signingKey = getSigningKeyFromJwks(jwksUri, token);
			if (signingKey == null) {
				return new ResponseMessage("JWT decode failed: Signing key not found");
			}

			// Validate and decode JWT
			SignedJWT signedJWT = SignedJWT.parse(token);
			JWTClaimsSet claimsSet = signedJWT.getJWTClaimsSet();

			return new ResponseMessage("Hello!", claimsSet.toJSONObject());

		} catch (Exception e) {
			return new ResponseMessage("JWT decode failed: " + e.getMessage());
		}
	}

	private static String fetchJwksUri(String baseAddress) throws Exception {
		String docUrl = baseAddress + "/.well-known/openid-configuration";
		HttpURLConnection conn = (HttpURLConnection) new URL(docUrl).openConnection();
		conn.setRequestMethod("GET");

		try (BufferedReader reader = new BufferedReader(new InputStreamReader(conn.getInputStream()))) {
			StringBuilder response = new StringBuilder();
			String line;
			while ((line = reader.readLine()) != null) {
				response.append(line);
			}

			String jwksUri = new org.json.JSONObject(response.toString()).getString("jwks_uri");
			if (jwksUri == null || jwksUri.isEmpty()) {
				throw new Exception("Failed to fetch JWKS URI.");
			}
			return jwksUri;
		}
	}

	private static RSAKey getSigningKeyFromJwks(String jwksUri, String token) throws Exception {
		JWKSet jwkSet = JWKSet.load(new URL(jwksUri));
		List<com.nimbusds.jose.jwk.JWK> keys = jwkSet.getKeys();

		SignedJWT signedJWT = SignedJWT.parse(token);
		String keyId = signedJWT.getHeader().getKeyID();

		return keys.stream()
				.filter(jwk -> jwk.getKeyID().equals(keyId))
				.findFirst()
				.map(jwk -> (RSAKey) jwk)
				.orElse(null);
	}

	static class ResponseMessage {
		public String message;
		public Object user_data;

		public ResponseMessage(String message) {
			this.message = message;
		}

		public ResponseMessage(String message, Object user_data) {
			this.message = message;
			this.user_data = user_data;
		}
	}
}

```
{% endtab %}

{% tab title="PHP" %}
**Step 1: Install Packages**

First, install the dependencies required by running these com

```bash
composer require firebase/php-jwt
```

```bash
composer require guzzlehttp/guzzle
```

**Step 2: Find the JWKS Endpoint**

Create a function that finds the JWKS endpoint from your Authgear application endpoint using the following code:

```php
<?php
require 'vendor/autoload.php';
use Firebase\JWT\JWT;

use Firebase\JWT\JWK;
use Firebase\JWT\Key;
use GuzzleHttp\Client;

$appUrl = ""; //place your authgear app endpoint here

function getJwksUri($appUrl) {
    $configEndpoint = $appUrl . "/.well-known/openid-configuration";
    $httpClient = new Client();
    $response = $httpClient->request('GET', $configEndpoint);
    $responseObject = json_decode($response->getBody());
    return $responseObject->jwks_uri;
}
```

**Step 3: Get Signing Key**

Add the following code to your application to get the JWT signing key:

```php
$jwksUri = getJwksUri($appUrl);
$httpClient = new Client();
$jwksUriResponse = $httpClient->request('GET', $jwksUri);
$keysObject = json_decode($jwksUriResponse->getBody());

$jwks = (array) ($keysObject->keys)[0];

$parsedKey = JWK::parseKey($jwks, "RS256");
$signingKey = $parsedKey->getKeyMaterial();
```

**Step 4: Extract the JWT From the Request Header**

To extract the access token from the HTTP request use the following code:

```php
if (!isset($_SERVER['HTTP_AUTHORIZATION']))
    throw new Exception("Invalid authorization header");
$authorizationHeader = $_SERVER['HTTP_AUTHORIZATION'];
$jwt = (explode(" ", $authorizationHeader))[1];
```

**Step 5: Validate and Decode JWT**

Finally, decode the JWT signing key.

```php
$decoded = JWT::decode($jwt, new Key($signingKey, 'RS256'));
echo json_encode($decoded);
```
{% endtab %}
{% endtabs %}

### Check the validity of JWT

The `auth_time` claim in an **OIDC ID token** represents the time **when the user authentication occurred**. Extract the `auth_time` claim from the token, which should represent the time of the original authentication in seconds. If the difference between the current time and `auth_time` exceeds your threshold (for example, 5 minutes), initiate the [re-authentication](../../authentication-and-access/authentication/reauthentication.md) process.

See an example of how to verify the signature of the ID token, and then validate the claims `auth_time` inside [here](../../authentication-and-access/authentication/reauthentication.md#backend-integration).

## Decode user from cookies

Validating JWT in your application server is _currently_ only available for **Token-based authentication**.

{% hint style="info" %}
For Cookie-based authentication, JWT in cookies is not supported yet. [You can track the issue here](https://github.com/authgear/authgear-server/issues/1180).
{% endhint %}
