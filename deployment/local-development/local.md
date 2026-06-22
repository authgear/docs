---
description: How to run Authgear locally with Docker Compose.
---

# Run locally with Docker Compose

Authgear is available as a Docker image. It depends on PostgreSQL (with pg\_partman enabled) and Redis.

To make it easy to try Authgear on your own machine, we provide an example repository with a complete Docker Compose setup and step-by-step instructions.

{% hint style="info" %}
This example is meant for local development and trying things out. It ships with default credentials and `DEV_MODE` turned on, so it works out of the box but isn't safe to expose to the internet.

Production deployments aren't off the table if you know what you're doing, but the defaults aren't ready to ship as-is. At minimum, swap out the credentials in the `env` file (database passwords, secret keys, JWT secrets), set the origins to your real domain, and turn off `DEV_MODE`.

For most production setups, we recommend [Deploy with Helm chart](../production-deployment/helm.md) instead.
{% endhint %}

**Clone the example repo and follow the included** `README.md`: \
[https://github.com/authgear/authgear-example-docker-compose](https://github.com/authgear/authgear-example-docker-compose)

The repository contains:

* Preconfigured `docker-compose.yml`
* Example environment files
* Guidance for initializing the project and running the portal + backend
* Tips for troubleshooting local development setups
