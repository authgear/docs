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

Here is the sample code on how to generate the JWT with the private key.

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

{% tab title="Java" %}
The following example is a basic Spring Boot application.

First, install the [JJWT](https://github.com/jwtk/jjwt) package by adding the following to the dependency block:

```gradle
dependencies {
    implementation 'io.jsonwebtoken:jjwt-api:0.12.6'
    runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.12.6'
    runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.12.6'
}
```

Then, implement your Java code like this:

```java
import java.io.InputStream;
import java.security.KeyFactory;
import java.security.PrivateKey;
import java.security.spec.PKCS8EncodedKeySpec;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.Base64;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.core.io.ClassPathResource;
import org.springframework.util.FileCopyUtils;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import io.jsonwebtoken.Jwts;

@SpringBootApplication
@RestController
public class DemoApplication {

	private static PrivateKey privateKey;

	static {
		try {
			// Load the private key from the PEM file (assuming it's in the classpath)
			ClassPathResource resource = new ClassPathResource("key.pem"); // store your downloaded Admin API key.pem in
																			// the resources folder.
			InputStream inputStream = resource.getInputStream();
			byte[] keyBytes = FileCopyUtils.copyToByteArray(inputStream);
			String privateKeyPEM = new String(keyBytes, "UTF-8");

			// Clean PEM string (remove header, footer, and newlines)
			privateKeyPEM = privateKeyPEM
					.replace("-----BEGIN PRIVATE KEY-----", "")
					.replace("-----END PRIVATE KEY-----", "")
					.replaceAll("\\n", "");

			// Decode the PEM string to get the raw key bytes
			byte[] decodedKey = Base64.getDecoder().decode(privateKeyPEM);

			// Generate the private key object from the bytes
			KeyFactory keyFactory = KeyFactory.getInstance("RSA"); // Use RSA
			PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(decodedKey);
			privateKey = keyFactory.generatePrivate(keySpec);
		} catch (Exception e) {
			// Handle the exception appropriately (e.g., log, throw a more specific
			// exception)
			System.err.println("Error loading or parsing private key: " + e.getMessage());
			throw new RuntimeException("Failed to initialize private key", e);
		}
	}

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

	@GetMapping("/generateJwt")
	public String generateJwt() {

		String kid = ""; // The key ID for your Admin API Key. Can be found in
																// Authgear Portal under Advanced > Admin API.
		String projectId = ""; // Your Authgear project ID. You can find this in Authgear Portal.
		int expirationInMinutes = 15; // how long the JWT will be valid for.

		Date now = new Date();
		Date expiration = new Date(now.getTime() + 30 * expirationInMinutes * 1000);

		Map<String, Object> claims = new HashMap<>();
		claims.put("aud", projectId);
		claims.put("iat", now);
		claims.put("exp", expiration);
		String jwt = Jwts.builder()
				.header()
				.add("kid", kid)
				.and()
				.claims(claims)
				.signWith(privateKey, Jwts.SIG.RS256)
				.compact();

		return "{\"jwt\": \"" + jwt + "\"}";
	}
}
```
{% endtab %}

{% tab title="ASP.NET" %}
The following example shows an ASP.NET Core API.\
First, install the required dependency:

```sh
dotnet add package System.IdentityModel.Tokens.Jwt
```

Next, create the following field variables for your project's ID, `kid`, the path to your private key (.pem file), and the validity of the JWT.

```csharp
private const string ProjectId = ""; // Replace with actual project ID
private const string KeyId = ""; // Replace with key ID from the portal
private const int ExpiryMinutes = 5;
private const string PrivateKeyPath = "key.pem"; // Path to PEM file
```

Then, implement a `GetPrivateKey` method that will get the private key from your .pem file.

```csharp
private RSA GetPrivateKey(string privateKeyPem)
{
    var rsa = RSA.Create();
    rsa.ImportFromPem(privateKeyPem.ToCharArray());
    return rsa;
}
```

Implement a `GenerateJwt` method. This method will use the private key and `kid` to generate a JWT.

```csharp
private string GenerateJwt(RSA privateKey)
{
    DateTime now = DateTime.UtcNow;

    var securityKey = new RsaSecurityKey(privateKey)
    {
        KeyId = KeyId // Set the Key ID
    };

    var credentials = new SigningCredentials(securityKey, SecurityAlgorithms.RsaSha256);

    var descriptor = new SecurityTokenDescriptor
    {
        Audience = ProjectId,
        IssuedAt = now,
        Expires = now.AddMinutes(ExpiryMinutes),
        SigningCredentials = credentials
    };

    var tokenHandler = new JwtSecurityTokenHandler();
    var token = tokenHandler.CreateToken(descriptor);
    return tokenHandler.WriteToken(token);
}
```

Finally, in your route method, read the .pem file and get a private key, then call the `GenerateJwt()` method to generate a JWT.

```csharp
[HttpGet(Name = "Jwt")]
public IActionResult GenerateJwt()
{
    try
    {
        // Read private key from file
        string privateKeyPem = System.IO.File.ReadAllText(PrivateKeyPath);
        RSA privateKey = GetPrivateKey(privateKeyPem);

        // Generate JWT
        string token = GenerateJwt(privateKey);
        return Ok(new { token });
    }
    catch (Exception ex)
    {
        return BadRequest(new { message = $"Error generating JWT: {ex.Message}" });
    }
}
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
