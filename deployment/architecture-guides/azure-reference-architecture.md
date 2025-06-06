# Azure Reference Architecture

<figure><img src="../../.gitbook/assets/authgear-infra-azure.png" alt=""><figcaption></figcaption></figure>

([Link](https://oursky.notion.site/Authgear-Reference-Architecture-Public-Page-099f15d621784f9299c86a6dcf55bade) to download original FigJam file)

### Cloud Resources Requirements

<table data-full-width="false"><thead><tr><th width="188">Products</th><th width="309">Purposes</th><th></th></tr></thead><tbody><tr><td>Azure Kubernetes Services</td><td>Pods to run the applications, cache for user sessions</td><td>D2 v3 (2 vCPUs, 8GB RAM)<br>x 3 minimum for k8s</td></tr><tr><td>Azure Database for PostgreSQL</td><td>Store system settings, user profiles, audit logs</td><td>D2s v3 (2 vCore)<br>x 2</td></tr><tr><td>Azure Cache for Redis</td><td>Store user sessions and usage analytics</td><td>Require approximately 30kB per user. Please refer to <a data-mention href="../production-deployment/helm.md">helm.md</a></td></tr><tr><td>(Optional Components)</td><td><p>Azure Storage:</p><ul><li>Storage of the user profile images</li></ul><p>Managed Elasticsearch:</p><ul><li>To support search in Authgear admin portal</li></ul><p>Networking:</p><ul><li>Azure Front Door with WAF Policy</li><li>Azure Load Balancer</li></ul><p>CI/CD</p><ul><li>Container registry</li><li>Secret management</li></ul><p>Logging and Monitoring tools</p></td><td></td></tr></tbody></table>

{% hint style="info" %}
For projects expecting more than 10,000 users, ElasticSearch is recommended for optimal search performance. In addition, when using PostgreSQL search, the result will not include a total count of matching items.
{% endhint %}

