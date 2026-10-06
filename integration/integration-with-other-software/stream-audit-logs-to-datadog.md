---
description: >-
  Learn how to stream your Authgear project's audit logs to Datadog
---

# Stream Audit Logs to Datadog

Authgear can send your project's audit logs to Datadog, so you can search, alert on and keep them alongside the rest of your logs.

### What gets sent

Every entry written to your project's [audit log](../../admin/monitor/audit-log.md) is also sent to Datadog: sign-ups, logins, profile changes, Admin API and Portal actions, and so on. Entries are delivered in batches about once a minute.

Each log arrives with `source:authgear` and `service:authgear`, and with Datadog's standard attributes (such as user ID, IP address and user agent) already mapped. You don't need to set up a pipeline in Datadog.

### Prerequisites

1. An Authgear account
2. A Datadog account with Log Management enabled

## Step 1: Get a Datadog API key

In Datadog, go to **Organization Settings** > [**API Keys**](https://app.datadoghq.com/organization-settings/api-keys) and create a new key, or copy an existing one.

Make sure it is an **API key**, not an **application key**. Application keys cannot send logs.

A Datadog API key can write to every part of your Datadog organization, so treat it like a password.

## Step 2: Find your Datadog site

Datadog runs several regional sites, and Authgear needs to know which one your organization is on. Check the address you use to sign in to Datadog:

| Datadog address | Site |
| --- | --- |
| `app.datadoghq.com` | US1 (`datadoghq.com`) |
| `us3.datadoghq.com` | US3 (`us3.datadoghq.com`) |
| `us5.datadoghq.com` | US5 (`us5.datadoghq.com`) |
| `app.datadoghq.eu` | EU1 (`datadoghq.eu`) |
| `ap1.datadoghq.com` | AP1 (`ap1.datadoghq.com`) |
| `ap2.datadoghq.com` | AP2 (`ap2.datadoghq.com`) |
| `uk1.datadoghq.com` | UK1 (`uk1.datadoghq.com`) |
| `app.ddog-gov.com` | US1-FED (`ddog-gov.com`) |

See [Getting Started with Datadog Sites](https://docs.datadoghq.com/getting_started/site/) for the full list.

## Step 3: Connect Datadog in the Authgear Portal

1. Log in to the Authgear Portal, select your project and go to **Integrations**.
2. Click **Connect** next to **Datadog**.
3. Choose your **Datadog site**. If your site is not in the list, choose **Other** and enter the site, for example `us2.ddog-gov.com`.
4. Paste your **API key**.
5. Click **Save**.

The Datadog row now shows **Connected**.

Authgear does not check the key or the site when you save. If logs don't appear in Datadog within a few minutes, check that the API key and the site are correct.

## Step 4: View your logs in Datadog

Trigger some activity in your project, such as signing up a test user. Then open the [Logs Explorer](https://app.datadoghq.com/logs) in Datadog and search for:

```
source:authgear service:authgear
```

Logs appear within about a minute.

{% hint style="info" %}
You don't need to install the Datadog Agent. Authgear sends logs directly to Datadog's log intake, so you can skip Datadog's "Install your first Agent" setup steps.
{% endhint %}

## Change or disconnect

To change the site or the API key, go to **Integrations** and click **Edit** next to Datadog. To see or change the current API key, click **Edit** next to the key. You may be asked to sign in again first.

To stop sending logs, click **Edit** and then **Delete**. Authgear removes the stored API key as well.

## Self-hosted Authgear

If you run Authgear yourself, logs are delivered by the Authgear background worker (`authgear background`). Make sure it is running, or entries are queued but never sent.
