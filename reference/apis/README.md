---
description: >-
  Authgear exposes APIs for developers to manage their applications
  programmatically
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# APIs

Besides the Client SDKs, Authgear exposes the following APIs for simple integration with your applications for authentication and user management.

All of these are on the endpoint of your app. The default endpoint is at `https://[myapp].authgear.cloud` unless you set up a custom domain. `[myapp]` is your project name.

Unless otherwise specified, all paths mentioned here are relative to the endpoint of your app.

Authgear provides the following groups of APIs:

* **OAuth 2.0 and OpenID Connect**: for connecting with OIDC Clients
* **Admin API**: for your servers to manage users via a GraphQL endpoint.
* **User Import API:** this is an API that supports the bulk import of users from another system to an Authgear project.
* **Authentication Flow (Auth Flow) API**: for developing a customized Web or Mobile Native Auth UI instead of the default user interface provided by Authgear.
* **Resolver Endpoint**: for API Gateway or Servers to check the validity of access tokens or cookies in the request header.

Here are all of the special paths with each group of the API above.

## OAuth 2.0 and OpenID Connect

For more information about the OIDC API endpoint, please refer to the following sections or any of the Regular Web App Getting Started guides.

{% content-ref url="oauth-2.0-and-openid-connect-oidc/" %}
[oauth-2.0-and-openid-connect-oidc](oauth-2.0-and-openid-connect-oidc/)
{% endcontent-ref %}

{% content-ref url="../../how-to-guide/authenticate/oidc-provider.md" %}
[oidc-provider.md](../../how-to-guide/authenticate/oidc-provider.md)
{% endcontent-ref %}

The related URLs are:

* `/.well-known/openid-configuration`\
  \
  This endpoint serves as a JSON document containing the OpenID Connect configuration of your app. That includes the authorization endpoint, the token endpoint, and the JWKs endpoint. Here is [an example of how it looks](https://accounts.portal.authgear.com/.well-known/openid-configuration).
* `/.well-known/oauth-authorization-server`\
  \
  This endpoint serves a JSON document containing the authorization server metadata of your app. That includes the authorization endpoint, the token endpoint, and the JWKs endpoint. Here is [an example of how it looks](https://accounts.portal.authgear.com/.well-known/openid-configuration).
* `/oauth2/userinfo`\
  \
  The UserInfo Endpoint is an OAuth 2.0 Protected Resource that returns Claims about the authenticated end user. When the client presents with a valid Access Token, the endpoint responds with the claims packaged in a JSON object. The claims are also the attributes of the [User Profile](../../how-to-guide/user-profiles/user-profile.md).

## Admin API

For more details about the Admin API, please refer to the following documentation:

{% content-ref url="admin-api/" %}
[admin-api](admin-api/)
{% endcontent-ref %}

The path for the Admin API is:

* `/_api/admin/graphql`

## User Import API

To learn more about using the User Import API, see the following documentation page:

{% content-ref url="../../how-to-guide/user-management/import-users-using-user-import-api.md" %}
[import-users-using-user-import-api.md](../../how-to-guide/user-management/import-users-using-user-import-api.md)
{% endcontent-ref %}

The path for the User Import API:

* `/_api/admin/users/import`

Use this endpoint to import users.

* `/_api/admin/users/import/{ID}`

Use this endpoint to query the status of an existing user import task.

## Authentication Flow API

You can find a detailed overview of the Authentication Flow API in the following documentation:

{% content-ref url="../../how-to-guide/custom-ui/authentication-flow-api.md" %}
[authentication-flow-api.md](../../how-to-guide/custom-ui/authentication-flow-api.md)
{% endcontent-ref %}

The path for the Authentication Flow API is:

* `/api/v1/authentication_flows`

## Resolver Endpoints

The resolver endpoint is at the following URL:

* `/_resolver/resolve`

The endpoint serves as a resolver to check the access token or cookie in the request headers. Forward incoming HTTP requests to this endpoint and the resolver will add the `x-authgear-` headers to the response.

See the list of `x-authgear-` headers in the specs [here](https://github.com/authgear/authgear-server/blob/master/docs/specs/api-resolver.md).

See implementation examples [here](../../get-started/backend-integration/nginx.md).

Should you choose to use Resolver Endpoints instead of JWT tokens to validate each API request, check out the following tutorial to learn how to go about that:

{% content-ref url="../../get-started/backend-integration/nginx.md" %}
[nginx.md](../../get-started/backend-integration/nginx.md)
{% endcontent-ref %}

## Other Special URLs

Here are two other URLs

* `/`\
  This endpoint is the entry point of the Web UI. You can visit it if you want to try your configuration (only for custom domains). However, this is NOT the authorization endpoint. You must use our SDK to initiate an authentication.
* `/settings`

This URL points to the default User settings UI provided by Authgear.
