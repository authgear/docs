---
description: Measured login and signup throughput of a self-managed Authgear deployment, for infrastructure planning
---

# Throughput Scaling Reference

In a load test in October 2026, a self-managed Authgear deployment on the hardware below handled about **11 logins per second (40,000 per hour)** or **9 signups per second (32,000 per hour)**. None of the roughly 30,000 test logins and signups failed.

| Role                             | Machines | Each               |
| -------------------------------- | -------- | ------------------ |
| Kubernetes worker                | 2        | 8 vCPUs, 16 GB RAM |
| Kubernetes control plane         | 1        | 4 vCPUs, 16 GB RAM |
| PostgreSQL (primary and standby) | 2        | 2 vCPUs, 8 GB RAM  |
| Redis (master and replica)       | 2        | 2 vCPUs, 8 GB RAM  |
| Reverse proxy                    | 2        | 2 vCPUs, 4 GB RAM  |

The layout follows [K8S Cluster Reference Architecture](on-premises-reference-architecture.md).

## Notes

* The test used login and signup with a phone number and an SMS one-time code, about 7 HTTP requests per flow. SMS delivery time is not included.
* Under light load, a login takes about 0.6 seconds and a signup about 0.7 seconds. Once the deployment is at capacity, extra flows wait in line rather than fail: at three times the load that reaches capacity, a login takes about 2.6 seconds and a signup about 3.4 seconds.
* Plan on logins per hour at your busiest time, with headroom for peaks. These figures are measurements of one deployment, not guarantees.
* Different hardware, flows with more steps, or the hosted Auth UI will give different results. Load test your own deployment before relying on these figures.
* [Architecture Guides](README.md) lists the minimum requirements for a highly available deployment, which are smaller than the configuration tested here. For virtual machines without Kubernetes, see the expected capacity in [Docker Compose Reference Architecture](docker-compose-reference-architecture.md#expected-capacity).
