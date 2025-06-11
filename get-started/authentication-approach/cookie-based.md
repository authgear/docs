---
description: Authenticate incoming request by cookie in the HTTP header.
---

# Cookie-based (Website or Single-page app)

By using Authgear, you can add a login to your website easily. Authgear supports various authentication methods, that you can easily turn on and configure in the portal.

## Prerequisites

To authenticate with cookies, you will need to set up a custom domain for Authgear, so that your website and Authgear are under the same root domains. e.g. Your website is `yourdomain.com`, and Authgear with a custom domain `auth.yourdomain.com`.

In this setting, if you have multiple applications under `yourdomain.com`, all applications would share the same session cookies automatically.

## **Overview**

### **How it works**

Your app server will receive a request with the cookie

![](https://mermaid.ink/img/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gICAgcGFydGljaXBhbnQgQ2xpZW50QXBwIGFzIEJyb3dzZXJcbiAgICBwYXJ0aWNpcGFudCBBdXRoZ2VhciBhcyBBdXRoZ2VhclxuICAgIHBhcnRpY2lwYW50IEFwcEJhY2tlbmQgYXMgWW91ciBBcHAgU2VydmVyXG4gICAgQ2xpZW50QXBwLT4-QXV0aGdlYXI6IDEuIFVzZXIgYXV0aGVudGljYXRlcyB3aXRoIEF1dGhnZWFyXG4gICAgQXV0aGdlYXItPj5DbGllbnRBcHA6IDIuIEF1dGhnZWFyIHNldHMgY29va2llXG4gICAgQ2xpZW50QXBwLT4-QXBwQmFja2VuZDogMy4gUmVxdWVzdCB3aXRoIGNvb2tpZVxuICAgIEFwcEJhY2tlbmQtPj5BcHBCYWNrZW5kOiA0LiBWZXJpZnkgUmVxdWVzdFxuICAgIEFwcEJhY2tlbmQtPj5DbGllbnRBcHA6IDUuIFNlcnZlciByZXNwb25kcyB3aXRoIHRoZSByZXF1ZXN0ZWQgaW5mb3JtYXRpb25cbiAgICAgICAgICAgICIsIm1lcm1haWQiOnt9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)

### **Verify request in your app server**

To verify the requests in your app server, you must **Forward authentication to Authgear Resolver Endpoint.**

![Forwarding authentication to Authgear Resolver Endpoint](https://mermaid.ink/img/eyJjb2RlIjoiZmxvd2NoYXJ0IFREXG4gICAgYXV0aGdlYXJbQXV0aGdlYXJdXG4gICAgYXBwW1lvdXIgQXBwIFNlcnZlcl1cbiAgICBcbiAgICBhcHAgLS0-IHwgRm9yd2FyZCBhdXRoZW50aWNhdGlvbiB0byA8YnIvPiBBdXRoZ2VhciByZXNvbHZlciBlbmRwb2ludCB8IGF1dGhnZWFyXG4iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)

## Request example

```javascript
> GET /api_path HTTP/1.1
> Host: yourdomain.com
> cookie: session=<AUTHGEAR_SESSION_ID>
```

## Get Started <a href="#get-started" id="get-started"></a>

The following tutorials show you how to add user login to your website using Authgear.

### 1. Frontend Integration

{% content-ref url="../single-page-app/website.md" %}
[website.md](../single-page-app/website.md)
{% endcontent-ref %}

### 2. Backend Integration

{% content-ref url="../backend-api/backend-integration.md" %}
[backend-integration.md](../backend-api/backend-integration.md)
{% endcontent-ref %}