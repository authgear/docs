---
description: Measured login and signup throughput of a self-managed Authgear deployment, for infrastructure planning
---

# Throughput Scaling Reference

Use this page to estimate the infrastructure a self-managed Authgear deployment needs for a given rate of logins and signups. The figures come from a load test of one on-premises Kubernetes deployment, run in October 2026. They are measurements of that deployment, not guarantees. A deployment with different hardware, network, or authentication flows will perform differently.

## Tested deployment

Authgear 2026-10-08.0 ran on a Kubernetes cluster of virtual machines, with the main server at 2 replicas. The deployment follows the layout in [K8S Cluster Reference Architecture](on-premises-reference-architecture.md): Authgear on Kubernetes, PostgreSQL with a standby, and Redis with a replica.

| Role                             | Machines | Each               |
| -------------------------------- | -------- | ------------------ |
| Kubernetes worker                | 2        | 8 vCPUs, 16 GB RAM |
| Kubernetes control plane         | 1        | 4 vCPUs, 16 GB RAM |
| PostgreSQL (primary and standby) | 2        | 2 vCPUs, 8 GB RAM  |
| Redis (master and replica)       | 2        | 2 vCPUs, 8 GB RAM  |
| Reverse proxy (Apache)           | 2        | 2 vCPUs, 4 GB RAM  |

The PostgreSQL standby and the Redis replica exist for failover and serve no traffic, so one PostgreSQL machine and one Redis machine did all the datastore work during the test.

## Test method

The test used [k6](https://grafana.com/docs/k6/latest/) from a machine outside the deployment's network. Requests reached Authgear through the reverse proxy and the cluster's Istio ingress, so the times below include the round trip over the internet.

Two flows were tested, each from the OAuth authorization request to the token exchange:

* **Login** with a phone number and an SMS one-time code.
* **Signup** with a phone number and an SMS one-time code.

Each flow makes about 7 HTTP requests: the authorization request, the [Authentication Flow API](../../reference/apis/authentication-flow-api.md) calls for each step, the finish redirect, and the token request. The flows call the API directly, as a custom UI does. These figures do not cover the hosted Auth UI, which renders an HTML page for each step on top of the work measured here.

The one-time code was fixed, so no SMS was sent. SMS delivery time is not included, and the SMS gateway was under no load. The OTP request rate limit was turned off on the test project so that every flow could request a code.

Each run kept a fixed number of virtual users (VUs) busy for 5 minutes. A VU starts its next flow as soon as the previous one finishes, with no pause between steps. Runs were made at 1, 5, 10, 15, 20, and 30 VUs. Every request had a 10-second timeout, and a flow with any failed request or unexpected response counted as failed.

## Results

Flow time is the wall-clock time for one complete flow, from the authorization request to the token response.

### Login

| VUs | Flows completed | Flows per second | HTTP requests per second | Average flow time | P95 flow time | Failed |
| --- | --------------- | ---------------- | ------------------------ | ----------------- | ------------- | ------ |
| 1   | 530             | 1.8              | 12                       | 0.57 s            | 0.68 s        | 0      |
| 5   | 2,656           | 8.8              | 62                       | 0.57 s            | 0.72 s        | 0      |
| 10  | 3,413           | 11.3             | 79                       | 0.88 s            | 1.15 s        | 0      |
| 15  | 3,493           | 11.4             | 80                       | 1.29 s            | 1.69 s        | 0      |
| 20  | 3,542           | 11.6             | 82                       | 1.69 s            | 2.21 s        | 0      |
| 30  | 3,539           | 11.2             | 79                       | 2.55 s            | 3.33 s        | 0      |

### Signup

| VUs | Flows completed | Flows per second | HTTP requests per second | Average flow time | P95 flow time | Failed |
| --- | --------------- | ---------------- | ------------------------ | ----------------- | ------------- | ------ |
| 1   | 412             | 1.4              | 10                       | 0.73 s            | 0.89 s        | 0      |
| 5   | 2,135           | 7.1              | 50                       | 0.70 s            | 0.91 s        | 0      |
| 10  | 2,610           | 8.7              | 61                       | 1.15 s            | 1.50 s        | 0      |
| 15  | 2,681           | 8.9              | 62                       | 1.68 s            | 2.21 s        | 0      |
| 20  | 2,675           | 8.9              | 62                       | 2.24 s            | 2.95 s        | 0      |
| 30  | 2,688           | 8.9              | 62                       | 3.36 s            | 4.43 s        | 0      |

Individual HTTP requests stayed fast throughout. For login, the average request took 80 ms at 1 VU, 125 ms at 10 VUs, and 362 ms at 30 VUs.

## What the results show

* **Capacity.** The deployment completes about **11 logins per second** (about 40,000 per hour) or **9 signups per second** (about 32,000 per hour), which is about 80 HTTP requests per second. It reaches this rate at about 10 concurrent flows. Beyond that, more concurrency adds waiting time, not throughput.
* **Response time.** Up to 10 concurrent flows, a login takes 0.6 to 0.9 seconds and a signup 0.7 to 1.2 seconds, close to the single-user figures. At 30 concurrent flows, a login takes 2.6 seconds and a signup 3.4 seconds on average. The slowest single flow in the whole test took 6 seconds.
* **Reliability.** None of the roughly 30,000 flows failed, at any load level.
* **The limiting component was not identified.** The test measured the deployment from the outside and did not record CPU, memory, or database load on the servers. Do not assume that adding worker nodes raises the limit in proportion.

## Using these figures for planning

* **Plan on logins per second, not concurrent users.** A VU in this test is far busier than a real user, because it starts the next flow the moment one finishes. Real users spend seconds or minutes between steps, so this deployment serves far more than 10 real users at once. Estimate the number of logins in your busiest hour and divide by 3,600 to get the rate to plan for. Peaks within the hour are higher than the hourly average, so leave headroom.
* **Add SMS delivery time** to the flow times above to estimate what a user experiences.
* **Load test your own deployment** with your own flows before relying on these figures, especially if you use the hosted Auth UI, social login, or flows with more steps.
* [Architecture Guides](README.md) lists the minimum requirements for a highly available deployment, which are smaller than the configuration tested here. For deployments on virtual machines without Kubernetes, [Docker Compose Reference Architecture](docker-compose-reference-architecture.md#expected-capacity) gives expected login rates for its minimum and recommended specifications.
