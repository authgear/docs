# M2M Tokens

In M2M authorization, the access token issued to the applications takes the **JWT** (JSON Web Token) format. A typical payload of the access tokens issued to M2M applications looks like this:

```json
{
  "aud": [
    "https://myapi.com/api"
  ],
  "client_id": "b892697a2075af58",
  "exp": 1755013218,
  "iat": 1755011418,
  "iss": "https://myproject.authgear.cloud",
  "jti": "b89bf5e5261f26ed220491ebf0f991ff89b274a21c88350221683cd02b74c364",
  "scope": "read:orders write:orders",
  "sub": "client_id_b892697a2075af58"
}
```

* `aud`: The identifier(s) of the API resource(s) that this token is intended for. The API server should check this claim and ensure that it matches its own identifier before accepting the token.
* `client_id`: The unique identifier of the M2M application or client that requested the token. This helps the API server to determine which application is making the request and apply the correct authorization policies.
* `exp`: The _expiration time_ of the token, represented as a Unix timestamp in seconds. After this time, the token is no longer valid and must not be accepted by the API.
* `iat`: The _issued at_ time, also a Unix timestamp in seconds. This indicates when the token was issued and can be used to evaluate its freshness.
* `iss`: The issuer of the token, usually the Authgear project endpoint (such as `https://myproject.authgear.cloud`). The API server should verify this value matches the expected issuer to ensure the token is from a trusted source.
* `jti`: The JWT ID, a unique identifier for the token. This can be used for token revocation or to guard against replay attacks.
* `scope`: A space-delimited list of permissions granted to the client. The API server should check these scopes to determine what actions the client is authorized to perform (e.g., `read:orders`, `write:orders`).
* `sub`: The subject of the token. For M2M tokens, this generally identifies the client itself (formatted here as `client_id_{client_id}`).\


