---
description: >-
  Send your project's audit logs to Datadog, a SIEM or any syslog collector
---

# Audit Log Streaming

Audit Log Streaming sends your project's audit logs to your own log system shortly after each event happens. Use it to search and alert on Authgear activity alongside the rest of your logs, or to feed a SIEM such as Microsoft Sentinel or Splunk.

You can stream to:

* **Datadog**, set up in the Portal. See [Stream Audit Logs to Datadog](stream-audit-logs-to-datadog.md).
* **A syslog collector**, over TCP with optional TLS. This page covers it.

{% hint style="info" %}
On Authgear Cloud, Audit Log Streaming is not enabled by default. [Contact us](https://www.authgear.com/talk-with-us) to enable it for your project.
{% endhint %}

## What gets streamed

Every entry written to your project's [audit log](audit-log.md) is streamed: sign-ups, logins, profile changes, Admin API and Portal actions, and so on. See [Log events](audit-log.md#log-events) for the full list.

## How delivery works

* Entries are sent in batches once a minute, so each one reaches your collector within about a minute.
* Entries are sent in the order they happened.
* If a batch cannot be delivered, for example because your collector is down, it is not resent. The entries are still kept in the audit log, and you can fetch them with the [Admin API](audit-log.md#retrieve-with-admin-api).
* Up to 10,000 entries are queued between deliveries. If a project produces more than that in a minute, the oldest queued entries are dropped. They are still kept in the audit log.
* You can add more than one stream. Each stream receives every entry, and a failure of one does not affect the others.

## Stream to a syslog collector

### Add a stream

1. In the Portal, go to **Advanced** > **Edit Config**.
2. Add a stream under `telemetry.audit_logs.streams`:

```yaml
telemetry:
  audit_logs:
    streams:
    - name: collector
      type: syslog
      transport: tcp
      tcp:
        address: collector.example.com:6514
        tls:
          enabled: true
      syslog:
        format: rfc5424
        framing: newline
```

3. Click **Save**.

Your collector must be reachable from the internet at `tcp.address`.

### Settings

| Field | Required | Description |
| --- | --- | --- |
| `name` | Yes | A name for the stream, unique within the project. Letters, digits, `-` and `_`, up to 63 characters. |
| `type` | Yes | `syslog` |
| `transport` | Yes | `tcp` |
| `tcp.address` | Yes | Your collector, as `host:port`. |
| `tcp.tls.enabled` | No | `true` to connect over TLS. Off by default. |
| `syslog.format` | Yes | `rfc5424` |
| `syslog.framing` | Yes | How messages are separated: `newline` or `octet_counting`. Must match your collector. See [Framing](#framing). |
| `syslog.facility` | No | `user` or `local0` to `local7`. Defaults to `local0`. |
| `syslog.app_name` | No | The syslog APP-NAME. Defaults to `authgear`. |
| `syslog.structured_data_id` | No | The syslog SD-ID. Defaults to `authgear`. |

### TLS and mutual TLS

With `tcp.tls.enabled: true`, Authgear connects with TLS 1.2 or later and checks your collector's certificate against the host in `tcp.address`. A certificate from a public certificate authority works with no further setup.

If your collector uses a certificate from a private certificate authority, or requires Authgear to present a client certificate (mutual TLS), [contact us](https://www.authgear.com/talk-with-us) to add the certificates to your project.

### Message format

Each entry is one [RFC 5424](https://datatracker.ietf.org/doc/html/rfc5424) syslog message. For example:

```
<134>1 2026-07-27T10:42:36Z myproject.authgear.cloud authgear - authgear-audit-log [authgear app_id="myproject" id="00000000000a5a60" activity_type="user.authenticated" user_id="00000000-0000-0000-0000-000000000001" client_id="0000000000000000" ip_address="203.0.113.9"] {"id":"00000000000a5a60","seq":678496,"type":"user.authenticated","payload":{...},"context":{...}}
```

* **Hostname** is your project's domain.
* **Message ID** is always `authgear-audit-log`.
* **Structured data** carries the key fields: `app_id`, `id`, `activity_type`, `user_id`, `client_id` and `ip_address`. A field is left out when it is empty. You can filter on these without parsing the message body.
* **Message body** is the full event as single-line JSON, the same as the **Raw Event Log** of an entry in the Portal.
* **Severity** is Warning for `email.error`, `sms.error` and `whatsapp.error`, and Informational for every other event.

### Framing

| `framing` | Format | Notes |
| --- | --- | --- |
| `newline` | One message per line | Works for every entry. Use this unless your collector requires octet counting. |
| `octet_counting` | Each message prefixed with its length ([RFC 6587](https://datatracker.ietf.org/doc/html/rfc6587#section-3.4.1)) | Many collectors cap these messages at 8 KB by default. Some entries are larger, so raise the limit on your collector. |

A message is usually a few KB. A `user.authenticated` entry includes the whole user and session, so it can be larger.

## Collector examples

### OpenTelemetry Collector

Add a syslog receiver that matches your stream:

```yaml
receivers:
  syslog:
    protocol: rfc5424
    tcp:
      listen_address: 0.0.0.0:6514
      tls:
        cert_file: /path/to/collector.crt
        key_file: /path/to/collector.key
```

`protocol` must be `rfc5424`. With `framing: newline`, no further setting is needed. To turn the message body into fields, add a JSON parser on the body.

### Microsoft Sentinel

Sentinel receives syslog through a Linux log forwarder running the Azure Monitor Agent. Follow Microsoft's guide, [Ingest syslog and CEF messages to Microsoft Sentinel with the Azure Monitor Agent](https://learn.microsoft.com/en-us/azure/sentinel/connect-cef-syslog-ama), using the **Syslog via AMA** connector. Then:

* Make the forwarder reachable from the internet, and set up TLS on its `rsyslog` or `syslog-ng` daemon. Microsoft's installation script listens without TLS on port 514.
* In the data collection rule, collect the `local0` facility, or the facility you set in `syslog.facility`.
* Raise the daemon's maximum message size to at least 16 KB. `rsyslog` cuts messages at 8 KB by default.

## Self-hosted Authgear

If you run Authgear yourself:

* Streaming is available unless `authgear.features.yaml` turns it off with `telemetry.audit_logs.streaming.disabled: true`.
* Add streams to `authgear.yaml` as shown above.
* Certificates for a private certificate authority or mutual TLS go in `authgear.secrets.yaml`, matched to a stream by `stream_name`:

```yaml
secrets:
- key: telemetry.audit_logs.streams.tls
  data:
  - stream_name: collector
    client_certificate:
      certificate:
        pem: |
          -----BEGIN CERTIFICATE-----
          ...
          -----END CERTIFICATE-----
      key:
        kty: RSA
        ...
    certificate_authority:
      pem: |
        -----BEGIN CERTIFICATE-----
        ...
        -----END CERTIFICATE-----
```

  * `client_certificate` is the certificate and private key (as a JWK) that Authgear presents to your collector. Add it for mutual TLS.
  * `certificate_authority` is the certificate authority your collector's certificate is checked against, instead of the system's trusted list.
  * Either can be set without the other.
* Entries are delivered by the background worker (`authgear background`). Make sure it is running, or entries are queued but never sent.
* The worker delivers once a minute by default. Set the `AUDIT_LOG_STREAMING_INTERVAL` environment variable (for example `30s`) to change it.

## Troubleshooting

* **No logs arrive.** Check that `tcp.address` is correct and reachable from the internet, and that your collector listens on TCP at that port. Then trigger some activity, such as signing in, and wait a minute.
* **TLS connection fails.** Check that your collector's certificate matches the host in `tcp.address` and is not expired. For a private certificate authority or mutual TLS, [contact us](https://www.authgear.com/talk-with-us).
* **Some entries are missing or cut off.** Your collector may be dropping long messages. Raise its maximum message size, or switch to `framing: newline`.
* **Logs arrive but your SIEM shows nothing.** Check that its filter includes the facility in `syslog.facility` (`local0` by default).
