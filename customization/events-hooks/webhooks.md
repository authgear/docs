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
{% tab title="Python" %}
```python
import hmac
import hashlib
import secrets
from http.server import HTTPServer, BaseHTTPRequestHandler

# Obtain the secret in the portal.
SECRET = "SECRET"

def hmac_sha256_string(data: bytes, secret: bytes) -> str:
    """Returns the hex-encoded string of HMAC-SHA256 code of body using secret as key."""
    hasher = hmac.new(secret, data, hashlib.sha256)
    signature = hasher.digest()
    sig = signature.hex()
    return sig

class WebhookHandler(BaseHTTPRequestHandler):
    def do_POST(self):
        try:
            content_length = int(self.headers.get('Content-Length', 0))
            body = self.rfile.read(content_length)
            
            sig_in_header = self.headers.get('X-Authgear-Body-Signature', '').encode()
            sig = hmac_sha256_string(body, SECRET.encode()).encode()
            
            # Prefer constant time comparison over == operator.
            if not secrets.compare_digest(sig_in_header, sig):
                # The signature does not match
                # Do NOT trust the content of this webhook!!!
                print(f"Signature mismatch: {sig_in_header.decode()} != {sig.decode()}")
                self.send_response(401)
                self.end_headers()
                return
            
            # Continue your logic here.
            self.send_response(200)
            self.end_headers()
            
        except Exception as e:
            # Handle the error properly
            print(f"Error: {e}")
            self.send_response(500)
            self.end_headers()

def main():
    server = HTTPServer(('', 9999), WebhookHandler)
    print("Server starting on port 9999...")
    try:
        server.serve_forever()
    except KeyboardInterrupt:
        print("Server stopped")
        server.shutdown()

if __name__ == "__main__":
    main()
```
{% endtab %}
{% tab title="NodeJS" %}
```JavaScript
const crypto = require('crypto');
const http = require('http');

// Obtain the secret in the portal.
const SECRET = 'SECRET';

// HMACSHA256String returns the hex-encoded string of HMAC-SHA256 code of body using secret as key.
function hmacSHA256String(data, secret) {
    const hasher = crypto.createHmac('sha256', secret);
    hasher.update(data);
    const signature = hasher.digest();
    return signature.toString('hex');
}

// Constant time comparison to prevent timing attacks
function constantTimeCompare(a, b) {
    if (a.length !== b.length) {
        return false;
    }
    
    let result = 0;
    for (let i = 0; i < a.length; i++) {
        result |= a.charCodeAt(i) ^ b.charCodeAt(i);
    }
    return result === 0;
}

const server = http.createServer((req, res) => {
    if (req.method !== 'POST') {
        res.writeHead(405);
        res.end();
        return;
    }

    let body = [];
    
    req.on('data', (chunk) => {
        body.push(chunk);
    });
    
    req.on('end', () => {
        try {
            const bodyBuffer = Buffer.concat(body);
            
            const sigInHeader = req.headers['x-authgear-body-signature'] || '';
            const sig = hmacSHA256String(bodyBuffer, SECRET);
            
            // Prefer constant time comparison over == operator.
            if (!constantTimeCompare(sigInHeader, sig)) {
                // The signature does not match
                // Do NOT trust the content of this webhook!!!
                throw new Error(`${sigInHeader} != ${sig}`);
            }
            
            // Continue you r logic here.
            res.writeHead(200);
            res.end();
            
        } catch (error) {
            // Handle the error properly
            console.error('Error:', error.message);
            res.writeHead(500);
            res.end();
        }
    });
    
    req.on('error', (error) => {
        // Handle the error properly
        console.error('Request error:', error);
        res.writeHead(500);
        res.end();
    });
});

server.listen(9999, () => {
    console.log('Server starting on port 9999...');
});
```
{% endtab %}
{% tab title="Java" %}
```Java
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.HttpServer;

import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.InetSocketAddress;
import java.security.InvalidKeyException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Arrays;

public class TestServer {
    // Obtain the secret in the portal.
    private static final String SECRET = "SECRET";
    
    /**
     * Returns the hex-encoded string of HMAC-SHA256 code of body using secret as key.
     */
    public static String hmacSHA256String(byte[] data, byte[] secret) throws NoSuchAlgorithmException, InvalidKeyException {
        Mac hasher = Mac.getInstance("HmacSHA256");
        SecretKeySpec keySpec = new SecretKeySpec(secret, "HmacSHA256");
        hasher.init(keySpec);
        byte[] signature = hasher.doFinal(data);
        return bytesToHex(signature);
    }
    
    private static String bytesToHex(byte[] bytes) {
        StringBuilder result = new StringBuilder();
        for (byte b : bytes) {
            result.append(String.format("%02x", b));
        }
        return result.toString();
    }
    
    /**
     * Constant time comparison to prevent timing attacks
     */
    private static boolean constantTimeCompare(String a, String b) {
        if (a.length() != b.length()) {
            return false;
        }
        
        byte[] aBytes = a.getBytes();
        byte[] bBytes = b.getBytes();
        
        int result = 0;
        for (int i = 0; i < aBytes.length; i++) {
            result |= aBytes[i] ^ bBytes[i];
        }
        return result == 0;
    }
    
    static class WebhookHandler implements HttpHandler {
        @Override
        public void handle(HttpExchange exchange) throws IOException {
            if (!"POST".equals(exchange.getRequestMethod())) {
                exchange.sendResponseHeaders(405, -1);
                return;
            }
            
            try {
                // Read request body
                InputStream requestBody = exchange.getRequestBody();
                byte[] body = requestBody.readAllBytes();
                requestBody.close();
                
                String sigInHeader = exchange.getRequestHeaders().getFirst("X-Authgear-Body-Signature");
                if (sigInHeader == null) {
                    sigInHeader = "";
                }
                
                String sig = hmacSHA256String(body, SECRET.getBytes());
                
                // Prefer constant time comparison over == operator.
                if (!constantTimeCompare(sigInHeader, sig)) {
                    // The signature does not match
                    // Do NOT trust the content of this webhook!!!
                    System.err.println(sigInHeader + " != " + sig);
                    exchange.sendResponseHeaders(401, -1);
                    return;
                }
                
                // Continue your logic here.
                exchange.sendResponseHeaders(200, -1);
                
            } catch (Exception e) {
                // Handle the error properly
                System.err.println("Error: " + e.getMessage());
                exchange.sendResponseHeaders(500, -1);
            }
        }
    }
    
    public static void main(String[] args) throws IOException {
        HttpServer server = HttpServer.create(new InetSocketAddress(9999), 0);
        server.createContext("/", new WebhookHandler());
        server.setExecutor(null); // Use default executor
        
        System.out.println("Server starting on port 9999...");
        server.start();
    }
}
```
{% endtab %}
{% tab title="PHP" %}
```php
<?php
// Obtain the secret in the portal.
const SECRET = 'SECRET';

/**
 * Returns the hex-encoded string of HMAC-SHA256 code of body using secret as key.
 */
function hmacSHA256String($data, $secret) {
    $signature = hash_hmac('sha256', $data, $secret, true);
    return bin2hex($signature);
}

/**
 * Constant time comparison to prevent timing attacks
 */
function constantTimeCompare($a, $b) {
    if (strlen($a) !== strlen($b)) {
        return false;
    }
    
    $result = 0;
    for ($i = 0; $i < strlen($a); $i++) {
        $result |= ord($a[$i]) ^ ord($b[$i]);
    }
    return $result === 0;
}

function main() {
    // Only handle POST requests
    if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
        http_response_code(405);
        exit();
    }
    
    try {
        // Read request body
        $body = file_get_contents('php://input');
        
        $sigInHeader = $_SERVER['HTTP_X_AUTHGEAR_BODY_SIGNATURE'] ?? '';
        $sig = hmacSHA256String($body, SECRET);
        
        // Prefer constant time comparison over == operator.
        if (!constantTimeCompare($sigInHeader, $sig)) {
            // The signature does not match
            // Do NOT trust the content of this webhook!!!
            error_log("Signature mismatch: $sigInHeader != $sig");
            http_response_code(401);
            exit();
        }
        
        // Continue your logic here.
        http_response_code(200);
        
    } catch (Exception $e) {
        // Handle the error properly
        error_log("Error: " . $e->getMessage());
        http_response_code(500);
    }
}

// Run the webhook handler
main();
?>
```
{% endtab %}
{% endtabs %}
