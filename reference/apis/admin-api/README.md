---
description: The Admin API allows your server to manage users via a GraphQL endpoint.
---

# Admin API

The Admin API allows your server to manage users via a GraphQL endpoint. You can list users, search users, view user details, and many more. In fact, the user management part of the portal is built with the Admin API.

## The Admin API GraphQL endpoint

The Admin API GraphQL endpoint is at `/_api/admin/graphql`. For example, if your app is `myapp` , then the endpoint is `https://myapp.authgear.cloud/_api/admin/graphql` .

## Authentication of the API endpoint

Accessing the Admin API GraphQL endpoint requires your server to generate a valid JWT and include it as `Authorization` HTTP header.

See [authentication-and-security.md](authentication-and-security.md "mention") to learn how to access the Admin API securely.

## API Explorer

If you want to explore what the Admin API can do, you can visit the GraphiQL tool. The GraphQL schema can also be found there.

* Go to **Advanced** -> **Admin API**
* Click on the **GraphiQL tool** link
* Toggle the **schema** documentation by pressing the Docs button in the top left corner.

<figure><img src="../../../.gitbook/assets/GraphiQL Explorer.png" alt=""><figcaption><p>Explor the Admin API with GraphiQL tool from the Admin Portal</p></figcaption></figure>

{% hint style="danger" %}
The GraphiQL tool is NOT a sandbox environment and all changes will be made on **real, live, production data**. Use with care!
{% endhint %}

## API Schema

{% content-ref url="api-schema.md" %}
[api-schema.md](api-schema.md)
{% endcontent-ref %}

## API Examples

{% content-ref url="api-queries-and-mutations.md" %}
[api-queries-and-mutations.md](api-queries-and-mutations.md)
{% endcontent-ref %}
