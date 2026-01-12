---
description: >-
  IP Blocklist allows you to block incoming traffic based on IP addresses or
  geographic regions.
---

# IP Blocklist

**IP Blocklist** allows you to block incoming traffic based on **IP addresses** or **geographic regions**. This helps prevent unwanted access and reduces automated attacks such as credential stuffing and bot traffic.

When enabled, requests from blocked IPs or regions are denied before authentication is processed.

### What Is IP Blocklist?

IP Blocklist is an access control mechanism that evaluates the client’s IP address at request time and blocks requests that match configured rules.

You can block traffic by:

* Specific IP addresses
* IP ranges using CIDR notation
* Geographic regions based on GeoIP

### Blocking IP Addresses

You can block individual IP addresses or ranges.

* Supports **IPv4**, **IPv6**, and **CIDR notation**
* Enter one IP address or range per line
* Useful for blocking known malicious sources or internal policy restrictions

Example:

```
192.0.2.10/32
203.0.113.0/24
2001:db8:85a3:0:0:8a2e:370:7334/128
```

### Blocking by Geographic Region

You can block traffic based on the user’s geographic location. GeoIP blocking is based on IP geolocation and may not be perfectly accurate.

### Checking an IP Address

The **Check IP address** tool lets you verify whether a specific IP address would be blocked by the current configuration. This is useful for testing new rules before deployment.

### Common Use Cases

* Blocking known malicious IP ranges
* Reducing bot traffic from high-risk regions
* Enforcing geographic access restrictions
* Responding quickly to active attacks
