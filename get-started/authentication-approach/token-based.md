---
description: Authenticate incoming request by access token in the HTTP header.
---

# Token-based (Native mobile or Single-page app)

By using Authgear, you can add the login feature to your mobile native app and single-page application easily. Authgear supports various authentication methods, that you can easily turn on and configure in the portal.

## **Overview**

### **How it works**

Your app server will receive a request with the access token

![](../../.gitbook/assets/token-based-authentication.png)

### Verify request in your app server

To verify the request in your app server, you can choose to **Forward authentication to Authgear Resolver Endpoint** or **Verify JSON Web Token (JWT) in your app server.**

![Forwarding authentication to Authgear Resolver Endpoint](https://mermaid.ink/img/eyJjb2RlIjoiZmxvd2NoYXJ0IFREXG4gICAgYXV0aGdlYXJbQXV0aGdlYXJdXG4gICAgYXBwW1lvdXIgQXBwIFNlcnZlcl1cbiAgICBcbiAgICBhcHAgLS0-IHwgRm9yd2FyZCBhdXRoZW50aWNhdGlvbiB0byA8YnIvPiBBdXRoZ2VhciByZXNvbHZlciBlbmRwb2ludCB8IGF1dGhnZWFyXG4iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)

![Verify JSON Web Token (JWT) in your app server](https://mermaid.ink/img/eyJjb2RlIjoiZmxvd2NoYXJ0IFREXG4gICAgYXBwW1lvdXIgQXBwIFNlcnZlcl1cbiAgICBcbiAgICBhcHAgLS0-IHxWZXJpZnkgSldUIHRva2VuIHwgYXBwXG4iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)

## Request Example

```bash
> GET /api_path HTTP/1.1
> Host: yourdomain.com
> Authorization: Bearer <AUTHGEAR_ACCESS_TOKEN>
```

## Get Started

The following tutorials show you how to add user login to your native mobile or single-page app using Authgear.

### 1. Frontend Integration

Choose your platform below:

{% content-ref url="../single-page-app/website.md" %}
[website.md](../single-page-app/website.md)
{% endcontent-ref %}

{% content-ref url="../native-mobile-app/react-native.md" %}
[react-native.md](../native-mobile-app/react-native.md)
{% endcontent-ref %}

{% content-ref url="../native-mobile-app/android/" %}
[android](../native-mobile-app/android/)
{% endcontent-ref %}

{% content-ref url="../native-mobile-app/ios.md" %}
[ios.md](../native-mobile-app/ios.md)
{% endcontent-ref %}

{% content-ref url="../native-mobile-app/flutter.md" %}
[flutter.md](../native-mobile-app/flutter.md)
{% endcontent-ref %}

{% content-ref url="../native-mobile-app/xamarin.md" %}
[xamarin.md](../native-mobile-app/xamarin.md)
{% endcontent-ref %}

### 2. Backend Integration

{% content-ref url="../backend-api/backend-integration.md" %}
[backend-integration.md](../backend-api/backend-integration.md)
{% endcontent-ref %}
