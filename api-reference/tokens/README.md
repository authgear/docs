---
description: >-
  Description of what's in Authgear's JWT access token and UserInfo endpoint
  response.
---

# Tokens

Authgear uses industry-standard tokens to manage user authentication and authorization. This section covers the different types of tokens, their structure, and how to use them in your applications.

## Token Types

Authgear provides several types of tokens for different purposes:

- **JWT Access Tokens**: Used for authorizing API calls and accessing protected resources
- **Refresh Tokens**: Used to obtain new access tokens when they expire
- **ID Tokens**: Contain user identity information for OpenID Connect flows

## Token Contents

Get details about what claims are returned in the JWT access token and response from the UserInfo endpoint by Authgear.

{% content-ref url="jwt-access-token.md" %}
[jwt-access-token.md](jwt-access-token.md)
{% endcontent-ref %}

{% content-ref url="refresh-token.md" %}
[refresh-token.md](refresh-token.md)
{% endcontent-ref %}
