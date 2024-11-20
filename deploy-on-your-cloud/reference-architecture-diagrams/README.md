# Reference Architecture Diagrams

Authgear Enterprise Plan offers self-managed options for customers to run the whole application on their private cloud or public cloud instances using our docker images. This is generally useful for regulated industries such as Financial, Insurance or Public Sectors which must retain the processing and storage of user data on-premises or in specific geographical area.

### Authgear Application Architecture Diagram

<figure><img src="../../.gitbook/assets/authgear-app-arch.png" alt=""><figcaption></figcaption></figure>

([Link](https://oursky.notion.site/Authgear-Reference-Architecture-Public-Page-099f15d621784f9299c86a6dcf55bade) to download original FigJam file)

### Deployment Options

* Depending on [backend-integration.md](../../get-started/backend-api/backend-integration.md "mention") approach, Authgear Resolvers is not needed if your backend self-validate user cookies or JWT.
* Supports custom SMS and email gateways.
* Redis is required to store user session, its recommended to use managed redis so session data can be persistent and users can keep their session in event of redis restart.
* Optional storage services:
  * Cloud storage is optional and only needed if allow user to upload profile image, which is disabled by default.
  * Elasticsearch is optional and only needed to support user search in the Authgear admin portal.
* Please refer to [helm.md](../helm.md "mention") for deployment options and details.
