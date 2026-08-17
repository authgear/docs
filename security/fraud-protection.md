---
description: >-
  Detect and stop fraudulent traffic such as SMS pumping before it drives up
  your messaging costs.
---

# Fraud Protection

Fraud Protection detects and mitigates fraudulent activity in your authentication flows. It evaluates risk signals on each request, makes a decision (allow, flag, or block), and logs every decision with reason codes so you can see what happened and why.

The current release focuses on **SMS pumping detection**. More risk signals are on the roadmap; see [What's next](fraud-protection.md#whats-next).

Find Fraud Protection in **Authgear Portal** > **Attack Protection** > **Fraud Protection**.

## SMS Pumping Detection

SMS pumping is a fraud scheme where attackers use bots to request large volumes of SMS one-time passwords (OTPs) for phone numbers they control or profit from. The OTPs are never verified. The goal is to make you send as many messages as possible and inflate your SMS bill.

Fraud Protection detects SMS pumping by watching for its telltale patterns on every SMS OTP request:

* **Unverified OTPs by IP**: An unusually high ratio of unverified OTPs requested from the same IP address, within an hour or a day.
* **Unverified OTPs by phone country**: An unusually high ratio of unverified OTPs sent to phone numbers of the same country, within an hour or a day.
* **Phone countries by IP**: A single IP address requesting OTPs for phone numbers across an unusually large number of countries within a day.

When a request trips any of these signals, it is flagged (or blocked, depending on the [mode](fraud-protection.md#modes)) and logged with the reason codes.

## Modes

Fraud Protection runs in one of two modes:

* **Observe mode** (default): Risky requests are flagged and logged, but not blocked. Use this to monitor fraud signals and confirm that legitimate users are not affected before enforcing rules.
* **Protect mode**: Risky requests are blocked in real time.

{% hint style="info" %}
Start with Observe mode and review the Overview and Logs tabs for a while. Once you are confident the flagged traffic is genuinely fraudulent, switch to Protect mode.
{% endhint %}

## Settings

Configure Fraud Protection in the **Settings** tab:

* **Enable Fraud Protection**: Turn the feature on or off. When off, Authgear detects and logs nothing.
* **Mode**: Choose between Observe mode and Protect mode.
* **Allowlist Settings**: Requests matching an allowlist are never blocked, even in Protect mode. Use this to exempt known-good traffic, such as your office network or internal test numbers.
  * **IP Allowlist**: A list of IP addresses or CIDR ranges.
  * **Allow the following countries based on Geo IP**: Requests from these countries, by IP geolocation, are not blocked.
  * **Phone Number Allowlist**: A list of full phone numbers (e.g. `+85212345678`) or regular expression patterns.
  * **Allow the following countries based on phone country code**: Requests to phone numbers registered in these countries are not blocked.

Detection thresholds are managed by Authgear with sensible defaults, so it works out of the box without tuning. To change the defaults, [contact us](https://www.authgear.com/talk-with-us).

## Monitoring

Two tabs show what Fraud Protection is doing:

* **Overview** tab: Total SMS OTP requests with flagged and blocked counts, a requests-by-action chart over time, and breakdowns of top source IPs, SMS destinations by recipient phone country, and source IP locations.
* **Logs** tab: Every decision as a queryable event, with the timestamp, action, result (Allow, Flagged, or Blocked), reason codes, source IP and its country, and the target phone number and its country. Open an entry to inspect the details, including the raw event log.

## What's next

SMS pumping detection is the first phase of Fraud Protection. On the roadmap:

* More risk signals, such as device fingerprinting and other bot activity indicators.
* Challenges (e.g. CAPTCHA) as an enforcement option for risky requests, in addition to allow and block.
* Configurable risk levels and external risk signals.
