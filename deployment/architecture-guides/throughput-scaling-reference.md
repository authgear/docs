---
description: Measured login and signup throughput of a self-managed Authgear deployment, for infrastructure planning
---

# Throughput Scaling Reference

Use this page to estimate the infrastructure a self-managed Authgear deployment needs for a given rate of logins and signups. The figures come from a load test of one on-premises Kubernetes deployment in October 2026. They are measurements of that deployment, not guarantees.

## Capacity at a glance

| Flow   | Per second | Per hour |
| ------ | ---------- | -------- |
| Login  | 11         | 40,000   |
| Signup | 9          | 32,000   |

The deployment below reached these rates with about 10 flows running at once. More concurrency made flows wait longer but did not increase throughput. No flow failed at any load level.

## Tested deployment

Authgear 2026-10-08.0 on a Kubernetes cluster of virtual machines, with the main server at 2 replicas. The layout follows [K8S Cluster Reference Architecture](on-premises-reference-architecture.md).

| Role                             | Machines | Each               |
| -------------------------------- | -------- | ------------------ |
| Kubernetes worker                | 2        | 8 vCPUs, 16 GB RAM |
| Kubernetes control plane         | 1        | 4 vCPUs, 16 GB RAM |
| PostgreSQL (primary and standby) | 2        | 2 vCPUs, 8 GB RAM  |
| Redis (master and replica)       | 2        | 2 vCPUs, 8 GB RAM  |
| Reverse proxy                    | 2        | 2 vCPUs, 4 GB RAM  |

The standby and the replica serve no traffic, so one PostgreSQL machine and one Redis machine handled all the datastore load.

## Throughput and response time by load

Each row is a 5-minute run with that many flows running back to back.

| Concurrent flows | Logins per second | Average login time | Signups per second | Average signup time |
| ---------------- | ----------------- | ------------------ | ------------------ | ------------------- |
| 1                | 1.8               | 0.6 s              | 1.4                | 0.7 s               |
| 5                | 8.8               | 0.6 s              | 7.1                | 0.7 s               |
| 10               | 11.3              | 0.9 s              | 8.7                | 1.2 s               |
| 15               | 11.4              | 1.3 s              | 8.9                | 1.7 s               |
| 20               | 11.6              | 1.7 s              | 8.9                | 2.2 s               |
| 30               | 11.2              | 2.6 s              | 8.9                | 3.4 s               |

Up to 10 concurrent flows, response times stay close to the single-user figures. Past that, each extra flow adds waiting time. Even at 30, every flow completed, and the slowest took 6 seconds.

## How the test was run

* Login and signup with a phone number and an SMS one-time code, through the [Authentication Flow API](../../reference/apis/authentication-flow-api.md). Each flow is about 7 HTTP requests, from the authorization request to the token exchange. The hosted Auth UI was not tested.
* No real SMS was sent. SMS delivery time is not included in the response times.
* About 30,000 flows in total, with 0 failures.

## Using these figures

* **Plan on logins per second, not concurrent users.** A concurrent flow in this test starts the next login the moment one finishes, so it is far busier than a real user. Take the number of logins in your busiest hour, divide by 3,600, and leave headroom for peaks within the hour.
* **Add SMS delivery time** to the response times above to estimate what a user experiences.
* **Do not assume adding nodes scales throughput in proportion.** The test did not identify which component set the limit.
* **Load test your own deployment** before relying on these figures, especially with the hosted Auth UI, social login, or flows with more steps.
* [Architecture Guides](README.md) lists the minimum requirements for a highly available deployment, which are smaller than the configuration tested here. For virtual machines without Kubernetes, see the expected capacity in [Docker Compose Reference Architecture](docker-compose-reference-architecture.md#expected-capacity).
