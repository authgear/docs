---
description: Overview of Authgear ONCE
---

# Authgear ONCE

Authgear ONCE is an easy-install version of Authgear that can run on any Linux server.&#x20;

ONCE is easier to set up than deploying the open-source version of Authgear. Compared to Authgear Cloud, it offers more customization and eliminates monthly subscriptions.

Anyone can purchase a one-time license for Authgear ONCE and install it on any popular cloud provider or VPS and have their own instance of Authgear running in less than 10 Minutes. The steps are easy, and you don't need to be a cloud or DevOps expert to use ONCE.

## Getting Started with Authgear ONCE

You can get started with Authgear ONCE in these four (4) steps:

### 1. Get Authgear ONCE License

Visit [authgear.com](https://authgear.com/) to purchase a one-time license for ONCE. Once the purchase is complete, you'll receive an email with your license and instructions on how to use it.

Each license and each installation contains 1 Authgear project. To create multiple projects, you need to buy multiple licenses.

### 2. Set up a Linux Virtual Machine

To get started with Authgear ONCE, the first thing you need to do is create a Virtual Machine (VM) with your preferred cloud provider.  Your VM should run a Linux server.&#x20;

Also, the machine should have Docker installed and have a public IP address. We recommend you create a machine that has Docker pre-installed if your cloud provider supports that. For VMs without Docker pre-installed, see [https://docs.docker.com/get-started/get-docker/](https://docs.docker.com/get-started/get-docker/) for guide on how to install Docker.

See our guides for setting up Authgear ONCE various cloud providers for more details:

{% content-ref url="authgear-once-installation-guide-for-vultr.md" %}
[authgear-once-installation-guide-for-vultr.md](authgear-once-installation-guide-for-vultr.md)
{% endcontent-ref %}

### 3. Configure Domain Name

A domain name is required to access your instance of Authgear ONCE on the internet. Get any domain name from your preferred registrar.

Then, add the public IP address for your VM as **A** records on your domain.

The following are the A records you should configure to get started with Authgear ONCE using the default sub-domains:

| Type | Name                     | Value         | Usage                                          |
| ---- | ------------------------ | ------------- | ---------------------------------------------- |
| A    | auth                     | IP of your VM | The authentication endpoint                    |
| A    | authgear-portal          | IP of your VM | The admin portal for CIAM functions            |
| A    | authgear-portal-accounts | IP of your VM | A domain for logging into the Authgear portal. |

### 4. Run the Installation Script

With an Authgear ONCE license, a VM with Docker, and a domain name with the correct A records ready, you can now proceed to run the ONCE installation script.

The installation process is interactive and only requires you to respond to a few prompts.&#x20;

The command for running the installation script is included in the email sent to you after the purchase of your ONCE license. Access your VM via SSH or a custom console your cloud provider offers, then paste the installation command. The command should look like this:

```sh
/bin/sh -c "$(curl -fsSL https://once-license.authgear.com/install/YOUR-AUTHGEAR-ONCE-LICENSE-KEY)"
```

Once the installation is complete, log in to `authgear-portal.your-domain.com`  to access the admin Portal for your new Authgear ONCE instance.

<figure><img src="../../.gitbook/assets/once-onboard.png" alt=""><figcaption><p>onboarding screen in Authgear Portal</p></figcaption></figure>

## Conclusion

Congratulations on installing your own instance of Authgear ONCE. You can continue reading the [official Authgear documentation site](https://docs.authgear.com/) to learn more about using all the features of Authgear.\
\
Contact support [here](https://www.authgear.com/schedule-demo), if you have any questions or need help with your setup.
