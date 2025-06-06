# Deploy with Helm chart

[authgear/helm-charts](https://github.com/authgear/helm-charts) is the recommended way to deploy Authgear on Kubernetes.

## Requirements

This section includes information about the software and the hardware requirements to run Authgear on Kubernetes.

### Kubernetes requirements

The minimum supported version of Kubernetes is 1.19.

### Storage requirements

Authgear does not store persist data on disk. It stores data in a PostgreSQL database and a Redis.

Authgear allows the end-user to upload their profile image. This feature is disabled by default. If you enable it, then Authgear requires a cloud object store. The supported cloud object store are AWS S3, GCP GCS, and Azure Blob Storage.

### CPU requirements

The CPU requirements depend on the number of users, workload and how active the users are. There are 4 scalable pods, 1 non-scalable pod and 1 images server in the basic setup. The scalable pods have a limit of `500m` CPU, the non-scalable one has `300m`, the images server has `1000m`. 2 Cores is recommended for the basic setup.

### Memory requirements

The scalable pods have a limit of `256MiB` of memory, the non-scalable one has `64MiB`, the images server have a limit of `1GiB` of memory. 1 GB of memory is recommended for the basic setup.

### Database requirements

PostgreSQL is the only supported database. PostgreSQL 12 is recommended. The PostgreSQL database must have the extension `pg_partman` installed, the version must be >= 4.0.

The database must have at least 5GB storage. The exact amount of storage depend on the number of users. About 100MB of storage is required to store 10,000 users.

Authgear stores its main data in a PostgreSQL database, and log data in another PostgreSQL database. 2 separate PostgreSQL databases are required. It is strongly recommended that the PostgreSQL databases are not shared with other software. The database account must have full access to the PostgreSQL database it connects to. Authgear uses the `public` schema.

If you decided to use PostgreSQL as the search service, an additional database is required.

Do not make changes to the PostgreSQL databases, the schemas, the tables, the columns, or the rows.

### Redis requirements

Authgear stores user sessions and other ephemeral data in Redis. The requirement is roughly 30kB per user. The recommended version of Redis is >= 6.2.

### Elasticsearch requirements

Authgear portal provides the search feature with Elasticsearch or PostgresSQL. If you decided to use Elasticsearch, a minimal setup of Elasticsearch consists of 3 Elasticsearch nodes is required. Each node requires 1 Core of CPU and 2GB of memory.

{% hint style="info" %}
For projects expecting more than 10,000 users, ElasticSearch is recommended for optimal search performance. In addition, when using PostgreSQL search, the result will not include a total count of matching items.
{% endhint %}

### Web browser requirements

Authgear supports the following web browsers:

* Apple Safari
* Google Chrome
* Microsoft Edge
* Mozilla Firefox

The latest two major versions of the supported browsers are supported.

### Hardware requirements summary

* CPU: 2 Cores CPU for the k8s nodes; 3 Cores for Elasticsearch
* Memory: 1GB memory for the k8s nodes; 6GB memory for Elasticsearch
* PostgreSQL 12 with `pg_partman>=4.0`, at least 5GB storage
* Redis >= 6.2, with 30kB per user. 10000 users require 300MB.

## How to install this Helm chart

This section provides detailed steps on how to install this Helm chart.

### Preparation on your local machine

You need to install the following tools on your local machine.

* `kubectl` with a version matching the Kubernetes server version. For example, if the server is 1.21, then you should be using the latest version of `kubectl` 1.21.x.
* Helm v3. You should use the latest version.
* Docker daemon. You need to be able to run Docker container on your local machine. If Docker daemon is unavailable, you need to download the binary release of Authgear to proceed. See [Download binary release](helm.md#down-binary-release) for details.

### Download binary release

> This step is optional if your local machine has a running Docker daemon.

If for some reason your local machine cannot have Docker daemon running, you can download the binary release of Authgear.

* Visit [https://github.com/authgear/authgear-server/releases](https://github.com/authgear/authgear-server/releases) to download the binary.
* You should choose a release closest to your intended version of Authgear.
* Currently, the binary is built for linux amd64 only.
* The name of the binary is in the format `authgear-lite-<platform>-<arch>-<tag>` and `authgear-portal-lite-<platform>-<arch>-<tag>`.
* You need to download both.

The following guide assumes you have downloaded the binary to your working directory and renamed them to `./authgear` and `./authgear-portal` respectively.

### Obtain a domain name

You need to obtain a domain name from a Internet domain registrar. If you already have a domain name, you can skip this step.

### Overview of the subdomains

This Helm chart assumes you have a apex domain dedicated to Authgear. Assume your apex domain is `myapp.com`.

Here is the list of subdomain assignments.

```
Authgear App ID  Domain                        Description
accounts         accounts.myapp.com            The default endpoint of the app "accounts"
accounts         accounts.portal.myapp.com     The custom domain endpoint of the app "accounts"
                 portal.myapp.com              The Authgear portal endpoint
app1             app1.myapp.com                The default endpoint of the app "app1"
...
...
```

### Provision the Kubernetes cluster

If you have a Kubernetes cluster already, you can skip creating a new one. Otherwise, follow the instructions from your cloud provider to create a new one. Refer to the [Hardware requirements summary](helm.md#hardware-requirements-summary) to configure the node pool.

### Provision the PostgreSQL database instance

> It is strongly recommended that you set up an external production-ready PostgreSQL instance, instead of relying on a in-cluster PostgreSQL deployment like [bitnami/postgresql](https://hub.docker.com/r/bitnami/postgresql).

If you have a PostgreSQL database instance already, you can skip creating a new one. Otherwise, follow the instructions from your cloud provider to create a new one. Refer to the [Database requirements](helm.md#database-requirements) to configure the instance.

Create 2 PostgreSQL databases within the instance. Create 1 PostgreSQL user for each PostgreSQL database. Make sure the PostgreSQL user has full access to the PostgreSQL database. See [Database requirements](helm.md#database-requirements) for details.

### Provision the Redis instance

> It is strongly recommended that you set up an external production-ready Redis instance, instead of relying on a in-cluster Redis deployment like [bitnami/redis](https://hub.docker.com/r/bitnami/redis).

If you have a Redis instance already, you can skip creating a new one. Otherwise, follow the instructions from your cloud provider to create a new one. Refer to the [Redis requirements](helm.md#redis-requirements) to configure the instance.

You should reserve 1 Redis database for Authgear.

### Provision the cloud object store

> This step is optional if you do not enable profile image.

Follow the corresponding guide of supported cloud object stores to create and configure.

For S3, Authgear needs the region, bucket name, access key ID and secret access key.

For GCS, Authgear needs the bucket name, service account and the credential JSON file.

For Azure Blob Storage, Authgear needs the storage account, container and access key.

It is recommended that you configure the object store to be non-public.

### Provision the SMTP server

If you have a SMTP server already, you can skip this step. Otherwise, you can subscribe to services such as SendGrid.

### Provision the NGINX ingress controller

If the Kubernetes cluster has NGINX ingress controller set up already, you can skip this step. Otherwise, you can use the Helm chart from [NGINX ingress controller](https://kubernetes.github.io/ingress-nginx/).

Note that Authgear expects the source IP of the incoming request to be correct. The source IP is used in rate limiting. If the source IP is incorrect, all requests are considered as coming the same source IP, making the limit being reached very soon.

One way correct the source IP is to set `externalTrafficPolicy` to `Local`. The caveat of this approach is that if the request is routed to a node without any NGINX ingress controller running on, the request is dropped. The simplest way to ensure one NGINX ingress controller running on a node is to use [DaemonSet](https://github.com/kubernetes/ingress-nginx/blob/helm-chart-4.0.17/charts/ingress-nginx/values.yaml#L191).

You need to change your DNS record so that all traffic of your domain go to the Kubernetes cluster.

### Provision the cert-manager

[cert-manager](https://cert-manager.io/docs/) automates the process of obtaining, renewing and using TLS certificates issued by [Let's Encrypt](https://letsencrypt.org/).

If you decide to manage TLS certificates by yourself, you can skip this step. Otherwise, you can use the Helm chart from [cert-manager](https://cert-manager.io/docs/installation/helm/)

Note that it is recommended that [you install the CRDs independent of the Helm chart](https://cert-manager.io/docs/installation/helm/#option-1-installing-crds-with-kubectl). The advantage of this approach is that the CRD resources can stay intact even if you uninstall the Helm chart.

### Create 2 namespaces

It is recommended to create these 2 namespaces.

* `authgear`: Install the helm chart in this namespace
* `authgear-apps`: Authgear-generated resources are in this namespace.

```
$ kubectl create namespace authgear
$ kubectl create namespace authgear-apps
```

### Create your own Helm chart

You need to create a few Kubernetes resources to support the Authgear Helm chart. So the best way is to create your own Helm chart and make the Authgear Helm chart a dependency.

#### Create the Helm chart

Create your Helm chart and then remove the generated boilerplate `.yaml` in the `templates/` directory.

```
$ helm create authgear-deploy
$ cd authgear-deploy/templates
$ rm -r test/ NOTES.txt deployment.yaml hpa.yaml ingress.yaml service.yaml serviceaccount.yaml
$ cd -
```

#### Add Authgear as dependency

Open `Chart.yaml` with your editor and make the following changes. The latest version can be found [here](https://github.com/authgear/helm-charts/releases).

```diff
--- a/authgear-deploy/Chart.yaml
+++ b/authgear-deploy/Chart.yaml
@@ -3,3 +3,7 @@ name: authgear-deploy
 type: application
 version: 0.1.0
 appVersion: 0.1.0
+dependencies:
+- name: authgear
+  version: "USE_LATEST_VERSION_HERE"
+  repository: "https://authgear.github.io/helm-charts"
```

Run the following to download dependencies.

```
$ helm dependency update ./authgear-deploy
```

#### Create cert-manager HTTP01 issuer and DNS01 issuer

You need to create a [HTTP01 issuer](https://cert-manager.io/docs/configuration/acme/http01/) and a [DNS01 issuer](https://cert-manager.io/docs/configuration/acme/dns01/) in both namespaces. So there are 4 issuers you need to create in total.

#### Run database migration

{% tabs %}
{% tab title="Docker" %}
```
$ docker run --rm -it quay.io/theauthgear/authgear-server authgear database migrate up \
  --database-url DATABASE_URL \
  --database-schema public
$ docker run --rm -it quay.io/theauthgear/authgear-server authgear images database migrate up \
  --database-url DATABASE_URL \
  --database-schema public
$ docker run --rm -it quay.io/theauthgear/authgear-server authgear audit database migrate up \
  --database-url DATABASE_URL \
  --database-schema public
$ docker run --rm -it quay.io/theauthgear/authgear-portal authgear-portal database migrate up \
  --database-url DATABASE_URL \
  --database-schema public
  
# Run this if you are using postgresql as search service
$ docker run --rm -it quay.io/theauthgear/authgear-portal authgear search migrate up \
  --search-database-url SEARCH_DATABASE_URL \
  --search-database-schema public
```
{% endtab %}

{% tab title="Binary" %}
```
$ ./authgear database migrate up \
  --database-url DATABASE_URL \
  --database-schema public
$ ./authgear images database migrate up \
  --database-url DATABASE_URL \
  --database-schema public
$ ./authgear audit database migrate up \
  --database-url DATABASE_URL \
  --database-schema public
$ ./authgear-portal database migrate up \
  --database-url DATABASE_URL \
  --database-schema public
  
# Run this if you are using postgresql as search service
$ ./authgear search database migrate up \
  --search-database-url SEARCH_DATABASE_URL \
  --search-database-schema public
```
{% endtab %}
{% endtabs %}

#### Create Elasticsearch index

> This step is optional if you do not enable Elasticsearch.

{% tabs %}
{% tab title="Docker" %}
```
$ docker run --rm -it quay.io/theauthgear/authgear-server authgear internal elasticsearch create-index \
  --elasticsearch-url ELASTICSEARCH_URL
```
{% endtab %}

{% tab title="Binary" %}
```
$ ./authgear internal elasticsearch create-index \
  --elasticsearch-url ELASTICSEARCH_URL
```
{% endtab %}
{% endtabs %}

#### Create deployment-specific authgear.secrets.yaml

Create a Secret that contains a `authgear.secrets.yaml` shared by all apps.

For example,

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: authgear-vendor-resources
type: Opaque
data:
  authgear.secrets.yaml: |-
    {{ include "authgear.authgearSecretsYAML" .Values.authgear | b64enc }}
```

#### Create the "accounts" app

Create the following directory structure

```
$ mkdir -p resources/authgear
```

Generate the `authgear.yaml` and `authgear.secrets.yaml`. Save the files to `resources/authgear` folder.

{% tabs %}
{% tab title="Docker" %}
```
$ docker run -v "$PWD"/resources:/app/resources --rm -it authgear-server authgear init \
  --output-folder=resources/authgear \
  --for-helm-chart
App ID (default 'my-app'): accounts
HTTP origin of authgear (default 'http://localhost:3000'): https://accounts.portal.myapp.com
HTTP origin of portal (default 'http://portal.localhost:8000'): https://portal.myapp.com
Phone OTP Mode (sms, whatsapp, whatsapp_sms) (default 'sms'):
Would you like to turn off email verification? (In case you don't have SMTP credentials in your initial setup) [Y/N] (default 'false'): N
```
{% endtab %}

{% tab title="Binary" %}
```
$ ./authgear init \
  --output-folder=resources/authgear \
  --for-helm-chart
App ID (default 'my-app'): accounts
HTTP origin of authgear (default 'http://localhost:3000'): https://accounts.portal.myapp.com
HTTP origin of portal (default 'http://portal.localhost:8000'): https://portal.myapp.com
Phone OTP Mode (sms, whatsapp, whatsapp_sms) (default 'sms'):
Would you like to turn off email verification? (In case you don't have SMTP credentials in your initial setup) [Y/N] (default 'false'): N
```
{% endtab %}
{% endtabs %}

Create the "accounts" app

{% tabs %}
{% tab title="Docker" %}
```
$ docker run -v "$PWD"/resources:/app/resources quay.io/theauthgear/authgear-portal authgear-portal internal configsource create ./resources/authgear \
  --database-url DATABASE_URL \
  --database-schema public

// FIXME: There isn't any easy way to configure a custom domain from cli
```
{% endtab %}

{% tab title="Binary" %}
```
$ ./authgear-portal internal setup-portal ./resources/authgear \
  --database-url DATABASE_URL \
  --database-schema public \
  --default-authgear-domain accounts.myapp.com \
  --custom-authgear-domain accounts.portal.myapp.com
```
{% endtab %}
{% endtabs %}

#### Prepare the values.yaml

Refer to [Helm chart values reference](helm.md#helm-chart-values-reference) and prepare the `./authgear-deploy/values.yaml`.

Remember to provide the correct client ID. The client ID can be found in the generated `authgear.yaml`.

#### Install your Helm chart

Install your helm chart with

```
helm install authgear-deploy ./authgear-deploy --namespace authgear --values ./authgear-deploy/values.yaml
```

## How to upgrade Authgear

If there are no breaking changes that require migration to be performed between the running version and the target version, an upgrade is as simple as setting `authgear.mainServer.image` and `authgear.portalServer.image` to a newer value.

If there are breaking changes, migration usually will be provided as a subcommand.

New features usually require database migration to add new tables and new columns. You may need to [run database migration](helm.md#run-database-migration) before you run `helm upgrade`. We try hard to make sure the modification to the database is backward-compatible, which means older version of Authgear can run with a higher version of database schema.

## Helm chart values reference

Please refer to [https://github.com/authgear/helm-charts/blob/master/authgear/values.yaml](https://github.com/authgear/helm-charts/blob/master/authgear/values.yaml)

| Name                                                                | Type    | Required | Description                                                                                                                                                                                                                 |
| ------------------------------------------------------------------- | ------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `authgear.appNamespace`                                             | String  | No       | The namespace to store Kubernetes resources created by Authgear. It is recommended to create a new namespace instead of reusing an existing one. You must create this namespace in advance. The default is `authgear-apps`. |
| `authgear.databaseURL`                                              | String  | Yes      | The database URL for Authgear to store its main data                                                                                                                                                                        |
| `authgear.databaseSchema`                                           | String  | Yes      | The database schema for Authgear to store its main data                                                                                                                                                                     |
| `authgear.redisURL`                                                 | String  | Yes      | The Redis URL for Authgear to store data with expiration, such as user sessions.                                                                                                                                            |
| `authgear.logLevel`                                                 | String  | No       | The log level                                                                                                                                                                                                               |
| `authgear.sentryDSN`                                                | String  | No       | The sentry DSN to report error logs                                                                                                                                                                                         |
| `authgear.ingress.enabled`                                          | Boolean | No       | Whether to create Ingresses according to the convention of this Helm chart                                                                                                                                                  |
| `authgear.ingress.class`                                            | String  | No       | The Ingress class. Only NGINX ingress controller is supported. The default is `nginx`                                                                                                                                       |
| `authgear.certManager.enabled`                                      | Boolean | No       | Whether cert-manager was installed by you and is available for this Helm chart to use. The default is `true`                                                                                                                |
| `authgear.certManager.issuer.dns01.name`                            | String  | Depends  | The name of the DNS01 issuer. It is required when cert-manager is enabled                                                                                                                                                   |
| `authgear.certManager.issuer.dns01.kind`                            | String  | Depends  | The kind of the DNS01 issuer. The default is `Issuer`                                                                                                                                                                       |
| `authgear.certManager.issuer.dns01.group`                           | String  | Depends  | The group of the DNS01 issuer. The default is `cert-manager.io`                                                                                                                                                             |
| `authgear.certManager.issuer.http01.name`                           | String  | Depends  | The name of the HTTP01 issuer. It is required when cert-manager is enabled                                                                                                                                                  |
| `authgear.certManager.issuer.http01.kind`                           | String  | Depends  | The kind of the HTTP01 issuer. The default is `Issuer`                                                                                                                                                                      |
| `authgear.certManager.issuer.http01.group`                          | String  | Depends  | The group of the HTTP01 issuer. The default is `cert-manager.io`                                                                                                                                                            |
| `authgear.baseHost`                                                 | String  | Yes      | The apex domain you assign to Authgear, for example `authgear.cloud`                                                                                                                                                        |
| `authgear.tls.wildcard.secretName`                                  | String  | No       | The name of the Secret to store the wildcard TLS certificate `*.baseHost`                                                                                                                                                   |
| `authgear.tls.portal.secretName`                                    | String  | No       | The name of the Secret to store the portal TLS certificate `portal.baseHost`                                                                                                                                                |
| `authgear.tls.portalAuthgear.secretName`                            | String  | No       | The name of the Secret to store the portal Authgear TLS certificate `accounts.portal.baseHost`                                                                                                                              |
| `authgear.smtp.host`                                                | String  | Yes      | The SMTP host                                                                                                                                                                                                               |
| `authgear.smtp.port`                                                | Integer | No       | The SMTP port. The default is `587`                                                                                                                                                                                         |
| `authgear.smtp.mode`                                                | String  | No       | The SMTP mode. Valid values are `normal` and `ssl`. When mode is `normal`, SSL usage is inferred from the port.                                                                                                             |
| `authgear.smtp.username`                                            | String  | No       | The SMTP username                                                                                                                                                                                                           |
| `authgear.smtp.password`                                            | String  | No       | The SMTP password                                                                                                                                                                                                           |
| `authgear.elasticsearch.enabled`                                    | Boolean | No       | Whether elasticsearch was deployed by you separately and is available for Authgear to use. The default is `false`                                                                                                           |
| `authgear.elasticsearch.url`                                        | String  | Depends  | The URL to the elasticsearch                                                                                                                                                                                                |
| `authgear.twilio.accountSID`                                        | String  | Depends  | The account SID of your Twilio subscription. It is required if you allow your users to authenticate with SMS. Either one of Twilio or Nexmo is enough                                                                       |
| `authgear.twilio.authToken`                                         | String  | Depends  | The auth token SID of your Twilio subscription.                                                                                                                                                                             |
| `authgear.nexmo.apiKey`                                             | String  | Depends  | The API key of your Nexmo subscription. It is required if you allow your users to authenticate with SMS. Either one of Twilio or Nexmo is enough                                                                            |
| `authgear.nexmo.apiSecret`                                          | String  | Depends  | The API secret of your Nexmo subscription.                                                                                                                                                                                  |
| `authgear.auditLog.enabled`                                         | Boolean | No       | Whether to make audit log available to view on the portal                                                                                                                                                                   |
| `authgear.auditLog.cronjob.enabled`                                 | Boolean | No       | Whether to enable the cronjob to run `pg_partman` procedure to create partitions                                                                                                                                            |
| `authgear.auditLog.cronjob.schedule`                                | String  | No       | The cron expression                                                                                                                                                                                                         |
| `authgear.auditLog.databaseURL`                                     | String  | Yes      | The database URL for Authgear to store its log data                                                                                                                                                                         |
| `authgear.auditLog.databaseSchema`                                  | String  | Yes      | The database schema for Authgear to store its log data                                                                                                                                                                      |
| `authgear.analytic.enabled`                                         | Boolean | No       | Whether to collect analytic data. The default is `false`                                                                                                                                                                    |
| `authgear.analytic.redisURL`                                        | String  | Yes      | The Redis URL for Authgear to store analytic data                                                                                                                                                                           |
| `authgear.mainServer.image`                                         | String  | Yes      | The Authgear server image                                                                                                                                                                                                   |
| `authgear.mainServer.resources`                                     | Object  | No       | Kubernetes ResourceRequirements for the main server                                                                                                                                                                         |
| `authgear.adminServer.resources`                                    | Object  | No       | Kubernetes ResourceRequirements for the admin API server                                                                                                                                                                    |
| `authgear.resolverServer.resources`                                 | Object  | No       | Kubernetes ResourceRequirements for the resolver server                                                                                                                                                                     |
| `authgear.background.resources`                                     | Object  | No       | Kubernetes ResourceRequirements for the background daemon                                                                                                                                                                   |
| `authgear.imagesServer.cdn.host`                                    | String  | No       | The CDN host for serving images                                                                                                                                                                                             |
| `authgear.imagesServer.objectStore.type`                            | String  | No       | The object store type. Valid values are `GCP_GCS`, `AWS_S3`, and `AZURE_BLOB_STORAGE`.                                                                                                                                      |
| `authgear.imagesServer.objectStore.awsS3.region`                    | String  | No       | The S3 region                                                                                                                                                                                                               |
| `authgear.imagesServer.objectStore.awsS3.bucketName`                | String  | No       | The S3 bucket name                                                                                                                                                                                                          |
| `authgear.imagesServer.objectStore.awsS3.accessKeyID`               | String  | No       | The S3 access key ID                                                                                                                                                                                                        |
| `authgear.imagesServer.objectStore.awsS3.secretAccessKey`           | String  | No       | The S3 secret access key                                                                                                                                                                                                    |
| `authgear.imagesServer.objectStore.gcpGCS.bucketName`               | String  | No       | The GCS bucket name                                                                                                                                                                                                         |
| `authgear.imagesServer.objectStore.gcpGCS.serviceAccount`           | String  | No       | The GCS service account. Typically in form of an email address.                                                                                                                                                             |
| `authgear.imagesServer.objectStore.gcpGCS.credentialsJSONContent`   | String  | No       | The content of the GCS credential JSON.                                                                                                                                                                                     |
| `authgear.imagesServer.objectStore.azureBlobStorage.storageAccount` | String  | No       | The name of the storage account                                                                                                                                                                                             |
| `authgear.imagesServer.objectStore.azureBlobStorage.container`      | String  | No       | The name of the container                                                                                                                                                                                                   |
| `authgear.imagesServer.objectStore.azureBlobStorage.accessKey`      | String  | No       | The access key                                                                                                                                                                                                              |
| `authgear.portalServerProxy.image`                                  | String  | No       | The Nginx sidecar image                                                                                                                                                                                                     |
| `authgear.portalServer.image`                                       | String  | Yes      | The Authgear portal server image                                                                                                                                                                                            |
| `authgear.portalServer.email.sender`                                | String  | No       | The email header Sender                                                                                                                                                                                                     |
| `authgear.portalServer.email.replyTo`                               | String  | No       | The email header Reply-To                                                                                                                                                                                                   |
| `authgear.portalServer.authgear.appID`                              | String  | Yes      | The app ID of the Authgear providing authentication for the portal server                                                                                                                                                   |
| `authgear.portalServer.authgear.clientID`                           | String  | Yes      | The client ID for the portal server to use Authgear                                                                                                                                                                         |
| `authgear.portalServer.authgear.endpoint`                           | String  | Yes      | The endpoint of the Authgear used by the portal server                                                                                                                                                                      |
| `authgear.portalServer.adminAPI.endpoint`                           | String  | Yes      | The static endpoint to the Admin API server. Normally this is an HTTP URL with cluster-local service name.                                                                                                                  |
| `authgear.portalServer.resources`                                   | Object  | No       | Kubernetes ResourceRequirements for the portal server                                                                                                                                                                       |
| `authgear.appCustomResources.path`                                  | String  | No       | The custom resources directory applied to every app. It provides global theming for this particular deployment.                                                                                                             |
| `authgear.appCustomResources.volume`                                | Object  | No       | Kubernetes Volume without the name field                                                                                                                                                                                    |
| `authgear.portalCustomResources.path`                               | String  | No       | The custom resources directory applied to the portal server. It provides theming for this particular deployment.                                                                                                            |
| `authgear.portalCustomResources.volume`                             | Object  | No       | Kubernetes Volume without the name field                                                                                                                                                                                    |

## Troubleshooting

### Duplicate Ingress definition

When you upgrade the Helm chart from v5 to v6, the Ingress admission controller will complain about duplicate Ingress definition. To resolve this problem, you have to manually delete the existing Ingress resources first. So the upgrade has downtime.

## Appendices

### Customize the subdomain assignment

This Helm chart has its own convention on the subdomain assignment and CANNOT be customized. If you want to customize the assignment, you can set `authgear.ingress.enabled` to `false`. You can then study the source code of this Helm chart, and create the Ingresses to suit your needs.
