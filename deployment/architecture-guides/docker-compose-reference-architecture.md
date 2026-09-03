---
description: >-
  Reference architecture for running Authgear on virtual machines with Docker
  Compose or Podman Compose.
---

# Docker Compose Reference Architecture

This page describes a reference architecture for running Authgear on conventional virtual machines with Docker Compose or Podman Compose. It suits deployments where a Kubernetes cluster is unavailable, or not worth operating for a single application. For the Kubernetes-based architecture, see [On-Premises Reference Architecture](on-premises-reference-architecture.md).

## Environments

Plan for two types of environment.

| Environment    | Purpose                                  | Availability                                                                |
| -------------- | ---------------------------------------- | --------------------------------------------------------------------------- |
| Non-production | Development, staging, and demonstration. | Single instance per component. Expect downtime during upgrades and reboots. |
| Production     | Live traffic.                            | Replicated components with no single point of failure.                      |

Non-production normally runs the cheaper non-HA setup. If staging must behave exactly like production, run the HA setup in both environments at the corresponding increase in cost.

## Components

The system has two major parts.

1. **Authgear**: the application services, their ingress (a reverse proxy), and the monitoring stack (Prometheus and Grafana), all run under Compose.
2. **Datastores**: PostgreSQL, Redis, and S3-compatible object storage.

## Choosing a Setup

Besides the environment type, one decision determines the setup: who runs the datastores.

* [**Option 1: Your Own Datastores**](#option-1-your-own-datastores). You supply PostgreSQL, Redis, and object storage from services you already operate. Choose this when your platform team already runs them. It is the preferred setup for production, because backup, patching, and failover stay with the team that already handles them.
* [**Option 2: Included Datastores**](#option-2-included-datastores). The datastores are installed on dedicated VMs as part of the deployment. Choose this when no managed datastores are available, such as air-gapped sites and bare-metal data centres, or when you want a self-contained, disposable environment.

|                                   | Non-production (no HA) | Production (HA)                    |
| --------------------------------- | ---------------------- | ---------------------------------- |
| **Option 1: your own datastores** | 1 VM                   | 2 VMs, plus an optional monitor VM |
| **Option 2: included datastores** | 2 VMs                  | 5 VMs                              |

Use the same option for both environments, so that non-production mirrors production's data layer.

### Expected Capacity

At 100 users, the expected sustained throughput is:

| Specification | Logins per second |
| ------------- | ----------------- |
| Minimum       | 5                 |
| Recommended   | 15                |

Throughput is bound by the database, not by the number of Authgear VMs, so the production setups differ from the non-production ones in availability alone. The production setups therefore fix Authgear at two VMs, enough for failover and rolling upgrades. A third would add nothing.

### Datastore Requirements

With Option 1 you supply the datastores. With Option 2 they are installed to the same requirements.

| Datastore      | Requirement                                        |
| -------------- | -------------------------------------------------- |
| PostgreSQL     | 16.14, with the `pg_partman` extension 16.5.1.     |
| Redis          | 6.2.20.                                            |
| Object storage | Any S3-compatible implementation.                  |

### High Availability

With Option 1, datastore availability is your responsibility. With Option 2, the non-production setup has no redundancy: each datastore is a single instance. The production setup provides it with [pg\_auto\_failover](https://github.com/hapostgres/pg_auto_failover) (PAF) for PostgreSQL and Sentinel for Redis, both driven from the monitor VM. The [High Availability](on-premises-reference-architecture.md#high-availability) section of the On-Premises Reference Architecture describes how these components work.

## Option 1: Your Own Datastores

You supply PostgreSQL, Redis, and S3-compatible object storage, and own their availability. The deployment adds only the VMs that run Authgear and its monitoring stack.

### Non-Production

Authgear, its ingress, and the monitoring stack share a single Docker host. That host is the only infrastructure the deployment adds, and the only single point of failure it introduces.

```mermaid
architecture-beta
  service user(internet)[User]
  service admin(internet)[Admin]
  service firewall(server)[Firewall]

  group Datastore(cloud)["Your Datastores"]
    service db(database)["PostgreSQL"] in Datastore
    service redis(database)["Redis"] in Datastore
    service s3(disk)["S3-Compatible Storage"] in Datastore
    junction j_datastore in Datastore
    j_datastore:T -- L:db
    j_datastore:R -- L:redis
    j_datastore:B -- L:s3

  group AuthgearVM(cloud)["Authgear VM"]
    group ComposeAuthgear(cloud) in AuthgearVM
      service ingress(server)[Ingress] in ComposeAuthgear
      service authgear(server)[Authgear] in ComposeAuthgear
      ingress:R --> L:authgear
      authgear:R -- L:j_datastore
      authgear:T --> B:prometheus

    group ComposeMonitor(cloud) in AuthgearVM
      service prometheus(server)[Prometheus] in ComposeMonitor
      service grafana(server)[Grafana] in ComposeMonitor
      prometheus:L -- R:grafana

  group External(cloud)[External Services]
    service mailer(server)[Mailer] in External
    service sms(server)["SMS Gateway"] in External
    service whatsapp(server)["WhatsApp"] in External
    service captcha(server)[CAPTCHA] in External
    junction j_mailer in External
    junction j_whatsapp in External
    junction j_sms in External
    junction j_captcha in External
    j_mailer:B -- T:mailer
    j_whatsapp:B -- T:whatsapp
    j_sms:B -- T:sms
    j_captcha:B -- T:captcha
    j_sms:L -- R:j_mailer
    j_sms:R -- L:j_whatsapp
    j_captcha:R -- L:j_mailer

user:R --> L:firewall
firewall:R --> L:ingress
admin:R --> L:grafana
authgear:B -- T:j_mailer
```

#### Inventory

You provide the following.

| Item          | Minimum                          | Recommended                      | Quantity | Remarks                                                                                                                                                                      |
| ------------- | -------------------------------- | -------------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IP address    |                                  |                                  | 1        |                                                                                                                                                                              |
| Domain        |                                  |                                  | 2 + P    | Two system domains for the Authgear Portal and Portal Login (for example `portal.example.com` and `accounts.example.com`), plus one Authgear Endpoint domain per project (P). |
| Certificate   |                                  |                                  | 1        | Wildcard certificate covering the domains above.                                                                                                                             |
| Firewall      |                                  |                                  | 1        |                                                                                                                                                                              |
| Authgear VM   | 2 CPU / 16 GB / 40 GB Data Disk  | 4 CPU / 16 GB / 100 GB Data Disk | 1        | Also runs the monitoring stack.                                                                                                                                              |
| Database      | 2 CPU / 16 GB / 40 GB Data Disk  | 4 CPU / 16 GB / 100 GB Data Disk | 1        | Your existing service. Size the disk to the expected number of users.                                                                                                        |
| Redis         | 1 CPU / 4 GB / 10 GB Data Disk   | 1 CPU / 8 GB / 10 GB Data Disk   | 1        | Your existing service.                                                                                                                                                       |
| S3            | 40 GB Storage                    | 80 GB Storage                    | 1        | Your existing service.                                                                                                                                                       |
| Mailer        |                                  |                                  | 1        | Optional. SMTP server or SendGrid.                                                                                                                                           |
| SMS           |                                  |                                  | 1        | Optional. Gateway account for SMS delivery.                                                                                                                                  |
| WhatsApp      |                                  |                                  | 1        | Optional. Business account for WhatsApp delivery.                                                                                                                            |
| CAPTCHA       |                                  |                                  | 1        | Optional. Google reCAPTCHA or Cloudflare Turnstile.                                                                                                                          |

### Production

Authgear runs on two VMs behind a load balancer, so either can be rebooted or upgraded without an outage. Monitoring moves to its own VM.

```mermaid
architecture-beta
  service user(internet)[User]
  service admin(internet)[Admin]
  service firewall(server)[Firewall]
  service lb(server)[Load Balancer]

  group Datastore(cloud)["Your Datastores"]
    service db(database)["PostgreSQL"] in Datastore
    service redis(database)["Redis"] in Datastore
    service s3(disk)["S3-Compatible Storage"] in Datastore
    junction j_datastore in Datastore
    j_datastore:T -- L:db
    j_datastore:R -- L:redis
    j_datastore:B -- L:s3

  group MonitorVM(cloud)["Monitor VM"]
    group ComposeMonitor(cloud) in MonitorVM
      service prometheus(server)[Prometheus] in ComposeMonitor
      service grafana(server)[Grafana] in ComposeMonitor
      prometheus:L -- R:grafana

  group AuthgearVM(cloud)["Authgear VM x2"]
    group ComposeAuthgear(cloud) in AuthgearVM
      service ingress(server)[Ingress] in ComposeAuthgear
      service authgear(server)[Authgear] in ComposeAuthgear
      ingress:R --> L:authgear
      authgear:R -- L:j_datastore
      authgear:T --> B:prometheus

  group External(cloud)[External Services]
    service mailer(server)[Mailer] in External
    service sms(server)["SMS Gateway"] in External
    service whatsapp(server)["WhatsApp"] in External
    service captcha(server)[CAPTCHA] in External
    junction j_mailer in External
    junction j_whatsapp in External
    junction j_sms in External
    junction j_captcha in External
    j_mailer:B -- T:mailer
    j_whatsapp:B -- T:whatsapp
    j_sms:B -- T:sms
    j_captcha:B -- T:captcha
    j_sms:L -- R:j_mailer
    j_sms:R -- L:j_whatsapp
    j_captcha:R -- L:j_mailer

user:R --> L:firewall
firewall:R --> L:lb
lb:R --> L:ingress
admin:R --> L:grafana
authgear:B -- T:j_mailer
```

#### Inventory

You provide the following.

| Item          | Minimum                          | Recommended                      | Quantity | Remarks                                                                                                                                                                      |
| ------------- | -------------------------------- | -------------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IP address    |                                  |                                  | 1        |                                                                                                                                                                              |
| Domain        |                                  |                                  | 2 + P    | Two system domains for the Authgear Portal and Portal Login (for example `portal.example.com` and `accounts.example.com`), plus one Authgear Endpoint domain per project (P). |
| Certificate   |                                  |                                  | 1        | Wildcard certificate covering the domains above.                                                                                                                             |
| Firewall      |                                  |                                  | 1        |                                                                                                                                                                              |
| Load balancer |                                  |                                  | 1        | Must be highly available itself.                                                                                                                                             |
| Authgear VM   | 2 CPU / 16 GB / 40 GB Data Disk  | 4 CPU / 16 GB / 100 GB Data Disk | 2        | For failover, not capacity.                                                                                                                                                  |
| Monitor VM    | 1 CPU / 8 GB / 40 GB Data Disk   | 1 CPU / 8 GB / 40 GB Data Disk   | 1        | Optional. Grafana dashboards over Prometheus.                                                                                                                                |
| Database      | 2 CPU / 16 GB / 40 GB Data Disk  | 4 CPU / 16 GB / 100 GB Data Disk | 1        | Your existing service. Size the disk to the expected number of users.                                                                                                        |
| Redis         | 1 CPU / 4 GB / 10 GB Data Disk   | 1 CPU / 8 GB / 10 GB Data Disk   | 1        | Your existing service.                                                                                                                                                       |
| S3            | 40 GB Storage                    | 80 GB Storage                    | 1        | Your existing service.                                                                                                                                                       |
| Mailer        |                                  |                                  | 1        | Optional. SMTP server or SendGrid.                                                                                                                                           |
| SMS           |                                  |                                  | 1        | Optional. Gateway account for SMS delivery.                                                                                                                                  |
| WhatsApp      |                                  |                                  | 1        | Optional. Business account for WhatsApp delivery.                                                                                                                            |
| CAPTCHA       |                                  |                                  | 1        | Optional. Google reCAPTCHA or Cloudflare Turnstile.                                                                                                                          |

## Option 2: Included Datastores

PostgreSQL, Redis, and S3-compatible object storage are installed on dedicated VMs as part of the deployment, to the [requirements above](#datastore-requirements). This is the only option that needs no managed datastore from you, at the cost of more VMs.

### Non-Production

The datastores run on one VM, and Authgear, its ingress, and the monitoring stack on a second. Every component is a single instance, so either host failing takes the service down. This is the cheapest setup and the quickest to stand up. Not for real users.

```mermaid
architecture-beta
  service user(internet)[User]
  service admin(internet)[Admin]
  service firewall(server)[Firewall]

  group DatastoreVM(cloud)["Datastore VM"]
    group Datastore(cloud) in DatastoreVM
      service db(database)["PostgreSQL"] in Datastore
      service redis(database)["Redis"] in Datastore
      service s3(disk)["S3-Compatible Storage"] in Datastore
      junction j_datastore in Datastore
      j_datastore:T -- L:db
      j_datastore:R -- L:redis
      j_datastore:B -- L:s3

  group AuthgearVM(cloud)["Authgear VM"]
    group ComposeAuthgear(cloud) in AuthgearVM
      service ingress(server)[Ingress] in ComposeAuthgear
      service authgear(server)[Authgear] in ComposeAuthgear
      ingress:R --> L:authgear
      authgear:R -- L:j_datastore
      authgear:T --> B:prometheus

    group ComposeMonitor(cloud) in AuthgearVM
      service prometheus(server)[Prometheus] in ComposeMonitor
      service grafana(server)[Grafana] in ComposeMonitor
      prometheus:L -- R:grafana

  group External(cloud)[External Services]
    service mailer(server)[Mailer] in External
    service sms(server)["SMS Gateway"] in External
    service whatsapp(server)["WhatsApp"] in External
    service captcha(server)[CAPTCHA] in External
    junction j_mailer in External
    junction j_whatsapp in External
    junction j_sms in External
    junction j_captcha in External
    j_mailer:B -- T:mailer
    j_whatsapp:B -- T:whatsapp
    j_sms:B -- T:sms
    j_captcha:B -- T:captcha
    j_sms:L -- R:j_mailer
    j_sms:R -- L:j_whatsapp
    j_captcha:R -- L:j_mailer

user:R --> L:firewall
firewall:R --> L:ingress
admin:R --> L:grafana
authgear:B -- T:j_mailer
```

#### Inventory

You provide the following. The datastores are installed on the database VM as part of the deployment.

| Item          | Minimum                          | Recommended                      | Quantity | Remarks                                                                                                                                                                      |
| ------------- | -------------------------------- | -------------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IP address    |                                  |                                  | 1        |                                                                                                                                                                              |
| Domain        |                                  |                                  | 2 + P    | Two system domains for the Authgear Portal and Portal Login (for example `portal.example.com` and `accounts.example.com`), plus one Authgear Endpoint domain per project (P). |
| Certificate   |                                  |                                  | 1        | Wildcard certificate covering the domains above.                                                                                                                             |
| Firewall      |                                  |                                  | 1        |                                                                                                                                                                              |
| Authgear VM   | 2 CPU / 16 GB / 40 GB Data Disk  | 4 CPU / 16 GB / 100 GB Data Disk | 1        | Also runs the monitoring stack.                                                                                                                                              |
| Database VM   | 2 CPU / 16 GB / 100 GB Data Disk | 4 CPU / 16 GB / 200 GB Data Disk | 1        | Runs PostgreSQL, Redis, and object storage. Size the disk to the expected number of users.                                                                                   |
| Mailer        |                                  |                                  | 1        | Optional. SMTP server or SendGrid.                                                                                                                                           |
| SMS           |                                  |                                  | 1        | Optional. Gateway account for SMS delivery.                                                                                                                                  |
| WhatsApp      |                                  |                                  | 1        | Optional. Business account for WhatsApp delivery.                                                                                                                            |
| CAPTCHA       |                                  |                                  | 1        | Optional. Google reCAPTCHA or Cloudflare Turnstile.                                                                                                                          |

### Production

Replicated PostgreSQL, Redis, and object storage run on two dedicated VMs. Authgear runs on two further VMs behind the load balancer, and the monitor VM carries the failover controllers: PAF for PostgreSQL and Sentinel for Redis. This is the largest footprint.

```mermaid
architecture-beta
  service user(internet)[User]
  service admin(internet)[Admin]
  service firewall(server)[Firewall]
  service lb(server)[Load Balancer]

  group DatastoreVM(cloud)["Datastore VM x2"]
    group Datastore(cloud) in DatastoreVM
      service db(database)["PostgreSQL"] in Datastore
      service redis(database)["Redis"] in Datastore
      service s3(disk)["S3-Compatible Storage"] in Datastore
      junction j_datastore in Datastore
      j_datastore:T -- L:db
      j_datastore:R -- L:redis
      j_datastore:B -- L:s3

  group MonitorVM(cloud)["Monitor VM"]
    group ComposeMonitor(cloud) in MonitorVM
      service prometheus(server)[Prometheus] in ComposeMonitor
      service grafana(server)[Grafana] in ComposeMonitor
      service paf(server)[PAF] in ComposeMonitor
      service sentinel(server)[Sentinel] in ComposeMonitor
      prometheus:L -- R:grafana
      prometheus:R -- L:paf
      paf:R -- L:sentinel

  group AuthgearVM(cloud)["Authgear VM x2"]
    group ComposeAuthgear(cloud) in AuthgearVM
      service ingress(server)[Ingress] in ComposeAuthgear
      service authgear(server)[Authgear] in ComposeAuthgear
      ingress:R --> L:authgear
      authgear:R -- L:j_datastore
      authgear:T --> B:prometheus

  group External(cloud)[External Services]
    service mailer(server)[Mailer] in External
    service sms(server)["SMS Gateway"] in External
    service whatsapp(server)["WhatsApp"] in External
    service captcha(server)[CAPTCHA] in External
    junction j_mailer in External
    junction j_whatsapp in External
    junction j_sms in External
    junction j_captcha in External
    j_mailer:B -- T:mailer
    j_whatsapp:B -- T:whatsapp
    j_sms:B -- T:sms
    j_captcha:B -- T:captcha
    j_sms:L -- R:j_mailer
    j_sms:R -- L:j_whatsapp
    j_captcha:R -- L:j_mailer

user:R --> L:firewall
firewall:R --> L:lb
lb:R --> L:ingress
admin:R --> L:grafana
authgear:B -- T:j_mailer
```

#### Inventory

You provide the following. The datastores are installed on the database VMs as part of the deployment.

| Item          | Minimum                          | Recommended                      | Quantity | Remarks                                                                                                                                                                      |
| ------------- | -------------------------------- | -------------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IP address    |                                  |                                  | 1        |                                                                                                                                                                              |
| Domain        |                                  |                                  | 2 + P    | Two system domains for the Authgear Portal and Portal Login (for example `portal.example.com` and `accounts.example.com`), plus one Authgear Endpoint domain per project (P). |
| Certificate   |                                  |                                  | 1        | Wildcard certificate covering the domains above.                                                                                                                             |
| Firewall      |                                  |                                  | 1        |                                                                                                                                                                              |
| Load balancer |                                  |                                  | 1        | Must be highly available itself.                                                                                                                                             |
| Authgear VM   | 2 CPU / 16 GB / 40 GB Data Disk  | 4 CPU / 16 GB / 100 GB Data Disk | 2        | For failover, not capacity.                                                                                                                                                  |
| Database VM   | 2 CPU / 16 GB / 100 GB Data Disk | 4 CPU / 16 GB / 200 GB Data Disk | 2        | Primary and replica. Size the disk to the expected number of users.                                                                                                          |
| Monitor VM    | 1 CPU / 8 GB / 40 GB Data Disk   | 1 CPU / 8 GB / 40 GB Data Disk   | 1        | Also runs the PAF and Sentinel failover controllers.                                                                                                                         |
| Mailer        |                                  |                                  | 1        | Optional. SMTP server or SendGrid.                                                                                                                                           |
| SMS           |                                  |                                  | 1        | Optional. Gateway account for SMS delivery.                                                                                                                                  |
| WhatsApp      |                                  |                                  | 1        | Optional. Business account for WhatsApp delivery.                                                                                                                            |
| CAPTCHA       |                                  |                                  | 1        | Optional. Google reCAPTCHA or Cloudflare Turnstile.                                                                                                                          |

## Network, Monitoring, and Backups

The firewall rules, hostnames, WAF paths, and backup guidance in [On-Premises Reference Architecture](on-premises-reference-architecture.md) apply to this architecture too. Where that page says Kubernetes, read the Authgear VMs.

Metrics are collected by the Prometheus and Grafana instances that ship with the deployment, on the Authgear VM in the non-production setups and on the monitor VM in the production setups. Access logs come from the ingress on each Authgear VM. Audit logs are persisted in PostgreSQL, so include the database in your backup policy.
