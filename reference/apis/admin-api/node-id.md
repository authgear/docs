# Using global node IDs

The node `id` is a globally unique identifier of an object. It is needed when you call the GraphQL query and mutation. In this section, we will go through how to generate the node ID for calling the Admin GraphQL API.

The node id is a [base64url](https://datatracker.ietf.org/doc/html/rfc4648#section-5) encoded string with the format of `<NODE_TYPE>:<ID>`. For example, in `User:97b1c929-842c-415c-a7df-6967efdda160` , the node is "User" while "97b1c929-842c-415c-a7df-6967efdda160" is the ID for a specific user.

## 1. Generate id for User node type

You can use your preferred programming language or tool to encode node IDs to base64url as shown below:

{% tabs %}
{% tab title="Go" %}
```go
package main

import (
	"encoding/base64"
	"fmt"
)

func main() {
	nodeID := base64.RawURLEncoding.EncodeToString([]byte("User:97b1c929-842c-415c-a7df-6967efdda160"))
	fmt.Println(nodeID)
}
```
{% endtab %}

{% tab title="Python" %}
```python
import base64

base64.urlsafe_b64encode(b'User:97b1c929-842c-415c-a7df-6967efdda160').replace(b'=', b'')
```
{% endtab %}

{% tab title="PHP" %}
```php
<?php
function base64url_encode($data) {
    return rtrim(strtr(base64_encode($data), '+/', '-_'), '=');
}

$encoded_id = base64url_encode("User:97b1c929-842c-415c-a7df-6967efdda160");
echo $encoded_id;

```
{% endtab %}

{% tab title="Node.js" %}
```javascript
const encoder = require('base64-url');

const encoded_id = encoder.encode("User:97b1c929-842c-415c-a7df-6967efdda160");
console.log(encoded_id);
```
{% endtab %}
{% endtabs %}

### Fetch the User object example

The query:

```graphql
query {
  node(id: "<BASE64URL_ENCODED_USER_NODE_ID>") {
    ... on User {
      id
      standardAttributes
    }
  }
}
```



## 2. Generate id for AuditLog node type&#x20;

{% tabs %}
{% tab title="Go" %}
```go
package main

import (
	"encoding/base64"
	"fmt"
)

func main() {
	nodeID := base64.RawURLEncoding.EncodeToString([]byte("AuditLog:000000000004d2e7"))
	fmt.Println(nodeID)
}
```
{% endtab %}

{% tab title="Python" %}
```python
import base64

base64.urlsafe_b64encode(b'AuditLog:000000000004d2e7').replace(b'=', b'')
```
{% endtab %}

{% tab title="PHP" %}
```php
<?php
function base64url_encode($data) {
    return rtrim(strtr(base64_encode($data), '+/', '-_'), '=');
}

$encoded_id = base64url_encode("AuditLog:000000000004d2e7");
echo $encoded_id;
```
{% endtab %}

{% tab title="Node.js" %}
```javascript
const encoder = require('base64-url');

const encoded_id = encoder.encode("AuditLog:000000000004d2e7");
console.log(encoded_id);
```
{% endtab %}
{% endtabs %}

### Fetch AuditLog object example

Query:

```graphql
{
  node(id: "<BASE64URL_ENCODED_AUDIT_LOG_NODE_ID>") {
    ... on AuditLog {
      id
      data
    }
  }
}
```

## 3. Generate id for Session node type

{% tabs %}
{% tab title="Go" %}
```go
package main

import (
	"encoding/base64"
	"fmt"
)

func main() {
	nodeID := base64.RawURLEncoding.EncodeToString([]byte("Session:8891767c-992c-4c09-bbae-9a8337ae661k"))
	fmt.Println(nodeID)
}
```
{% endtab %}

{% tab title="Python" %}
```python
import base64

base64.urlsafe_b64encode(b'Session:8891767c-992c-4c09-bbae-9a8337ae661k').replace(b'=', b'')
```
{% endtab %}

{% tab title="PHP" %}
```php
<?php
function base64url_encode($data) {
    return rtrim(strtr(base64_encode($data), '+/', '-_'), '=');
}

$encoded_id = base64url_encode("Session:8891767c-992c-4c09-bbae-9a8337ae661k");
echo $encoded_id;
```
{% endtab %}

{% tab title="Node.js" %}
```javascript
const encoder = require('base64-url');

const encoded_id = encoder.encode("Session:8891767c-992c-4c09-bbae-9a8337ae661k");
console.log(encoded_id);
```
{% endtab %}
{% endtabs %}

### Fetch Session object example

Query:

```graphql
{
  node(id: "<BASE64URL_ENCODED_SESSION_NODE_ID>") {
    ... on Session {
      id
      displayName
      type
      createdAt
    }
  }
}
```

## 4. Generate id for Authenticator node type

{% tabs %}
{% tab title="Go" %}
```go
package main

import (
	"encoding/base64"
	"fmt"
)

func main() {
	nodeID := base64.RawURLEncoding.EncodeToString([]byte("Authenticator:95435c90-8501-4735-a8b6-412601605a25"))
	fmt.Println(nodeID)
}
```
{% endtab %}

{% tab title="Python" %}
```python
import base64

base64.urlsafe_b64encode(b'Authenticator:95435c90-8501-4735-a8b6-412601605a25').replace(b'=', b'')
```
{% endtab %}

{% tab title="PHP" %}
```php
<?php
function base64url_encode($data) {
    return rtrim(strtr(base64_encode($data), '+/', '-_'), '=');
}

$encoded_id = base64url_encode("Authenticator:95435c90-8501-4735-a8b6-412601605a25");
echo $encoded_id;
```
{% endtab %}

{% tab title="Node.js" %}
```javascript
const encoder = require('base64-url');

const encoded_id = encoder.encode("Authenticator:95435c90-8501-4735-a8b6-412601605a25");
console.log(encoded_id);
```
{% endtab %}
{% endtabs %}

### Fetch Authenticator object example

Query:

```graphql
{
  node(id: "<BASE64URL_ENCODED_AUTHENTICATOR_NODE_ID>") {
    id
    ... on Authenticator {
      id
      kind
      createdAt
    }
  }
}
```

## 5. Generate id for Authorization node type

{% tabs %}
{% tab title="Go" %}
```go
package main

import (
	"encoding/base64"
	"fmt"
)

func main() {
	nodeID := base64.RawURLEncoding.EncodeToString([]byte("Authorization:12345a90-7801-41d5-c3b6-655501605b56"))
	fmt.Println(nodeID)
}
```
{% endtab %}

{% tab title="Python" %}
```python
import base64

base64.urlsafe_b64encode(b'Authorization:12345a90-7801-41d5-c3b6-655501605b56').replace(b'=', b'')
```
{% endtab %}

{% tab title="PHP" %}
```php
<?php
function base64url_encode($data) {
    return rtrim(strtr(base64_encode($data), '+/', '-_'), '=');
}

$encoded_id = base64url_encode("Authorization:12345a90-7801-41d5-c3b6-655501605b56");
echo $encoded_id;
```
{% endtab %}

{% tab title="Node.js" %}
```javascript
const encoder = require('base64-url');

const encoded_id = encoder.encode("Authorization:12345a90-7801-41d5-c3b6-655501605b56");
console.log(encoded_id);
```
{% endtab %}
{% endtabs %}

### Fetch Authorization object example

Query:

```graphql
{
  node(id: "<BASE64URL_ENCODED_AUTHORIZATION_NODE_ID>") {
    id
    ... on Authorization {
      id
      clientID
      scopes
      createdAt
    }
  }
}
```

## 6. Generate id for Identity node type

{% tabs %}
{% tab title="Go" %}
```go
package main

import (
	"encoding/base64"
	"fmt"
)

func main() {
	nodeID := base64.RawURLEncoding.EncodeToString([]byte("Identity:12345a90-7801-41d5-c3b6-655501605b56"))
	fmt.Println(nodeID)
}
```
{% endtab %}

{% tab title="Python" %}
```python
import base64

base64.urlsafe_b64encode(b'Identity:12345a90-7801-41d5-c3b6-655501605b56').replace(b'=', b'')
```
{% endtab %}

{% tab title="PHP" %}
```php
<?php
function base64url_encode($data) {
    return rtrim(strtr(base64_encode($data), '+/', '-_'), '=');
}

$encoded_id = base64url_encode("Identity:12345a90-7801-41d5-c3b6-655501605b56");
echo $encoded_id;
```
{% endtab %}

{% tab title="Node.js" %}
```javascript
const encoder = require('base64-url');

const encoded_id = encoder.encode("Identity:12345a90-7801-41d5-c3b6-655501605b56");
console.log(encoded_id);
```
{% endtab %}
{% endtabs %}

### Fetch Identity object example

Query:

```graphql
{
  node(id: "<BASE64URL_ENCODED_IDENTITY_NODE_ID>") {
    id
    ... on Identity {
      id
      claims
      type
    }
  }
}
```





