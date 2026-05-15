# Generate OTP code

Authgear Admin GraphQL API supports generating OTP code on behalf of end-user when they asks for support.

## API Details

The query:

```graphql
mutation ($target: String!) {
  generateOOBOTPCode(input: { target: $target }) {
    code
  }
}
```

The variables:

```json
{
  "target": "<USER_EMAIL_OR_PHONE_NUMBER>"
}
```

The sample variables:

```json
{
  "target": "user@example.com"
}
```

The sample response:

```json
{
  "data": {
    "generateOOBOTPCode": {
      "code": "123456"
    }
  }
}
```
