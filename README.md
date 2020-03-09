# Vault Production Readiness Checklist
This checklist will prepare you launch production-ready vault clusters into any major Cloud provider or on Premise

1. [Infrastructure Architecture](#Infrastructure-Architecture)
1. [Consul Storage Backend](#Consul-Storage-Backend)
1. [Load Balancing](#Load-Balancing)
1. [Monitoring and Alerting](#Monitoring-and-Alerting)
1. [Configuration Management](#Configuration-Management)
1. [Vault Configuration](#Vault-Configuration)
1. [Identity & Access Management](#Identity-&-Access-Management)
1. [Security Hardening](#Security-Hardening)
1. [Operational Readiness](#Operational-Readiness)
1. [Observability](#Disaster-Recovery)
1. [Governance and compliance](#Governance-and-compliance)


### **Infrastructure Architecture**

|  |  |
| --------- | ------- |
| &#9744;   | <details><summary>Infrastructure Workloads spread accross Availability Zones</summary> <p> Nodes in Vault clusters (and Consul clusters if being used as a storage backend) should be spread accross two or more failure domains known as Availability zones. The loss of a single Availability zone should not result result in a loss of service. </p> </details> |
| &#9744;   | <details><summary>Machine Images Created (Virtual Machines only)</summary> <p> If you are deploying your Vault nodes on virtual machines, It is reccomended to build re-usable VM images that can be used to create cluster nodes in an immutable way.  Tools like [Hashicorp Packer](https://packer.io/) are designed to help build repeatable machine images for most virtualised and cloud platform. Machine images should be versioned and should follow a release cycle as new images are produced.</p> </details> |
| &#9744;   | <details><summary>Replication Enabled (Enterprise only)</summary> <p> If you are using the Enterprise version of Vault, you can enable replication between two or more Vault clusters in different geographical regions for added protection is Disaster Recovery scenarios.  Replication can be configured in Disaster Recovery Mode or Performance Replication mode.  If you are planing on using Replication, you need to provision infrastructure in an alternative region, with nodes spread accross multiple Availability Zones. For more information about the Enterprise Replication feature, see [the official documentation.](https://www.vaultproject.io/docs/internals/replication/) </p> </details> |
| &#9744;   | <details><summary>Firewall rules configured to control access to Vault</summary> <p> Vault will likely contain business critical secrets which makes it a prime target for malicious actors. Access to vault to should be restricted to your private networks and not be accessible on the internet.  The Use of Virtual Private Networks is a commonly used approach to allow access to Vault from unknown networks</p> </details> |
| &#9744;   | <details><summary>Compute Resouces satisfy the minimum requirements</summary> <p> Ensure Hardware servers and Virtual Machines have been appropriately resources in accordance with the [Deployment System Requirements](https://learn.hashicorp.com/vault/operations/ops-reference-architecture#deployment-system-requirements) </p> </details> |
| &#9744;   | <details><summary>Secondary Disk</summary> <p>Ensure that vault servers have a secondary disk attached to them. This will help with Audit Device Fault tolerance</p> </details> |

### **Consul Storage Backend**

|  |  |
| --------- | ------- |
| &#9744;   | <details><summary>Storage Backend Architecture</summary> <p> </summary> <p>It is a recommended pattern to use [HashiCorp Consul's](https://www.consul.io/) Key/Value store as the storage backend. The reccomended cluster size for consul is 5 nodes.  This cluster size allows for fault tolerance whilst performing maintenence on a a node</p> </details> |
| &#9744;   | <details><summary>Consul Gossip Encryption configured</summary> <p> Members of the Consul clusters use a gossip protocol to communicate with eachother and hold leadership elections. This network traffic should be encrypted to minimise security risks.  You can read more about consul encryption [here.](https://www.consul.io/docs/agent/encryption.html) </p> </details> |
| &#9744;   | <details><summary>ACLs</summary> <p>The path that Vault uses in Consul's key/value storage to store it's encrypted data should be protected using Consul's ACL system. Once configured, the Management ACL token should be revoked.  You can read more about configuring Consul's ACL system [here.](https://www.consul.io/docs/acl/index.html)  </p> </details> |
| &#9744;   | <details><summary>Consul Backups scheduled</summary> <p>As Consul is being used as a data store that Vault uses, it should be considered a stateful service, and as such, should have a backup strategy.  Consul snapshot, in addition to disk backups should be implemented on a regular schedule. For more information about consul snapshot, click [here](https://www.consul.io/docs/commands/snapshot.html)</p> </details> |
| &#9744;   | <details><summary>Consul Clients installed on Vault Nodes</summary> <p>Vault should not talk directly to Consul backend as this introduces an increased attack vector.  Instead, Consul should be installed on the Vault servers and configured in client mode. The clients will facilitate the communication between Vault and Consul.</p> </details> |
| &#9744;   | <details><summary>Consul UI enabled (Optional)</summary> <p>If using a 5 node consul cluster, you can choose to enable the UI; however, it is recommended that the UI is enabled on two nodes only.</p> </details> |

### **Load Balancing**

|  |  |
| --------- | ------- |
| &#9744;   | <details><summary>TLS Encryption is configured on Load Balancer</summary> <p> </summary> <p> Vault’s communications should be encrypted end-to-end with TLS and this should not be terminated at the Load balancer layer. The load balancer should also use the same encryption to communicate with Vault</p> </details> |
| &#9744;   | <details><summary>HTTP Redirects configured</summary> <p> </summary> <p> With TLS configured, all traffic going via HTTPS will be encrypted; however, we need to ensure that there are no connections to vault via HTTP. The Load balancer should be configured to redirect all HTTP traffic to HTTPS.</p> </details> |
| &#9744;   | <details><summary>Health checks enabled and configured</summary> <p> </summary> <p>Load balancer health probes can be used to ensure that traffic is only routed to a healthy leader node. Configure routing rules according to [these response codes](https://www.vaultproject.io/api/system/health.html) </p> </details> |


### **Monitoring & Alerting**

|  |  |
| --------- | ------- |
| &#9744;   | <details><summary>Vault Telemetry Enabled.</summary> <p> </summary> <p>Vault telemetry should be configured in the telemetry stanza within the Vault config file. This will enable monitoring and alerting with a wide range of open source tools (Telegraf and prometheus)</p> </details> |
| &#9744;   | <details><summary>Consul Telemetry Enabled.</summary> <p> </summary> <p>Consul telemetry should be configured in the telemetry stanza within the consul config file. This will enable monitoring and alerting with a wide range of open source tools (Telegraf and prometheus)</p> </details> |
| &#9744;   | <details><summary>Vault platform monitoring configured</summary> <p> </summary> <p>Monitoring system of your choice is configured to monitor and alert on vault application metric thresholds as per the [best practice guidance of Hashicorp.](https://s3-us-west-2.amazonaws.com/hashicorp-education/whitepapers/Vault/Vault-Consul-Monitoring-Guide.pdf)</p> </details> |
| &#9744;   | <details><summary>Consul platform Monitoring Configured</summary> <p> </summary> <p>Monitoring system of your choice is configured to monitor and alert on infrastructure metric thresholds as per the [best practice guidance of Hashicorp.](https://s3-us-west-2.amazonaws.com/hashicorp-education/whitepapers/Vault/Vault-Consul-Monitoring-Guide.pdf)</p> </details> |
| &#9744;   | <details><summary>Infrastructure Monitoring Configured</summary> <p> </summary> <p>Monitoring system of your choice is configured to monitor and alert on consul application metric thresholds as per the [best practice guidance of Hashicorp.](https://s3-us-west-2.amazonaws.com/hashicorp-education/whitepapers/Vault/Vault-Consul-Monitoring-Guide.pdf)</p> </details> |
| &#9744;   | <details><summary>Monitoring Dashbaord Created</summary> <p> </summary> <p>Using a Dashboard tool a of your choice, create a monitoring dashboard for operations staff to easily identify any issues that may be occurring.</p> </details> |


### **Configuration Management**

|  |  |
| --------- | ------- |
| &#9744;   | <details><summary>Infrastructure as code written (Virtual Machines only)</summary> <p> </summary> <p> Code written to deploy the infrastructure for Consul and Vault. [Terrafrom](https://www.terraform.io/) is an appropriate tool for this task.  Virtual Machine images created from code for Consul and Vault. Packer is a good choice of tool for this. All Virtual infrastructure should be deployed and managed using an Infrastructure as code tool</p> </details> |
| &#9744;   | <details><summary>Vault Platform Configuration Code Written</summary> <p> </summary> <p> Vault Platform configuration should be described in code using a tool like [Terrafrom](https://www.terraform.io/).  Configuration such as Auth Methods, Secrets Engines, Audit Devices and Policies should all be configured using code</p> </details> |
| &#9744;   | <details><summary>Code under Version Control in Source Code Repositories</summary> <p> </summary> <p>All Infrastructure code and application code should be stored separate source control repositories and be placed under version control. An appropriate branching strategy should be implemented and documented in the README file.</p> </details> |
| &#9744;   | <details><summary>Code Owners in repositories</summary> <p> </summary> <p>Repository files should have code owners assigned to them to control who can approve Pull Requests that will be merged into the Master branch.</p> </details> |
| &#9744;   | <details><summary>Repository rules implemented</summary> <p> </summary> <p>Configure the minimum number of Pull Request approvers, restrictions on Pull Request Authors approving their own requests and any other rules that your organisation’s security standards require for Integrity.</p> </details> |
| &#9744;   | <details><summary>Deployment pipelines implemented</summary> <p> </summary> <p>Code deployments should be automated using deployment pipelines. Where possible, the pipeline should be written as code and stored under version control with the code</p> </details> |
| &#9744;   | <details><summary>Dev/Staging environments created</summary> <p> </summary> <p>Create development and staging environments for Vault.  Staging Environment should be identical to production, with the only divergence being, when pre-production changes are implemented for final testing prior to being deployed to production.</p> </details> |
| &#9744;   | <details><summary>Naming and coding standards established</summary> <p> </summary> <p>Implement and document naming and coding standards. Naming standards for Namespaces, Policies, Vault Roles, secrets keys and AppRoles.  Coding standards where applicable for variable names and function names.</p> </details> |
| &#9744;   | <details><summary>Integration tests written</summary> <p> </summary> <p>A suite of automated integration tests written to be run either during the deployment pipeline or as a pre-check on your chosen VCS required to pass before a Pull Request can be merged.</p> </details> |


### **Vault Configuration**

|  |  |
| --------- | ------- |
| &#9744;   | <details><summary>Enable audit device on 2 files on different disks</summary> <p> </summary> <p>Vault logs all requests and responses to requests. If Vault is unable to log requests and responses to these requests, it will immediately seize operations. To provide redundancy, each vault node should have 2 file audit devices enabled on separate volumes on separate disks.</p> </details> |
| &#9744;   | <details><summary>Log rotation configured on log files</summary> <p> </summary> <p>Enable and configure log rotation on the audit files to ensure the disks do not fill up and cause a vault outage.</p> </details> |
| &#9744;   | <details><summary>Configure at least one IDP as an auth method</summary> <p> </summary> <p>Where appropriate, configure an existing identity provider (or multiple if required) as an authentication method in Vault</p> </details> |
| &#9744;   | <details><summary>Configure the required secrets engines</summary> <p> </summary> <p>Identify and enable the required secrets engines for your business and technical use cases</p> </details> |
| &#9744;   | <details><summary>Ensure KV v2 is enabled</summary> <p> </summary> <p>Ensure that Version 2 of the KV secrets engine is used to enable secrets versioning</p> </details> |


### **Identity & Access Management**

|  |  |
| --------- | ------- |
| &#9744;   | <details><summary>Create default policies</summary> <p> </summary> <p>Create default policies that all user entities will inherit according to your business security model.  This could be list permissions on a particular KV path for example.</p> </details> |
| &#9744;   | <details><summary>Create Policy mappings for default policies</summary> <p> </summary> <p>Create a mapping for default policies to ensure all user entities inherit these policies.</p> </details> |
| &#9744;   | <details><summary>Configure aliases for entities when more than one auth method is in use</summary> <p> </summary> <p>Using the Identity Secrets engine, create aliases to attach vault logins via different auth methods to a single entity to ensure the correct policies are inherited and to make the logging data easier to mine</p> </details> |
| &#9744;   | <details><summary>Design a path structure for KV v2 that matches the way your org works (team based or product/service based)</summary> <p> </summary> <p>Map you KV path design to the way your organisation works or product groupings.</p> </details> |
| &#9744;   | <details><summary>Meta AppRole process defined</summary> <p> </summary> <p>Meta Approles are a mechanism that allow an application or service to read the secret id of an app role without exposing this to application developers.</p> </details> |


### **Security Hardening**

|  |  |
| --------- | ------- |
| &#9744;   | <details><summary>Ensure TLS is configured on Vault Cluster</summary> <p> </summary> <p>Enable end-to-end encryption using TLS certificates.  Vault agents should also use TLS certificates</p> </details> |
| &#9744;   | <details><summary>Ensure TLS is configured on Consul Clusters</summary> <p> </summary> <p>Enable end-to-end encryption on consul cluster and agent. More information can be found [here.](https://www.consul.io/docs/agent/encryption.html)</p> </details> |
| &#9744;   | <details><summary>Enable and configure SELinux / App Armour</summary> <p> </summary> <p>Enable and config SELinux / app amour depending on your operating system to create sandboxed contexts to  reduce blast radius if even the system is compromised.</p> </details> |
| &#9744;   | <details><summary>Randomise the ports used to differ from standard ports for Vault</summary> <p> </summary> <p>By default, Vault uses port 8200 and 8201. Change the port to a non-standard port to provide extra hardening</p> </details> |
| &#9744;   | <details><summary>Randomise the ports used to differ from standard ports for Consul</summary> <p> </summary> <p>By default, Consul uses port 8500 and 8501. Change the port to a non-standard port to provide extra hardening</p> </details> |
| &#9744;   | <details><summary>Revoke root token</summary> <p> </summary> <p>Once initial set-up of Vault cluster has been completed, the root token should be revoked.</p> </details> |
| &#9744;   | <details><summary>Revoke Consul Management ACL token</summary> <p> </summary> <p>Once the initial set-up of the Consul ACL system has been completed, the management token should be revoked.</p> </details> |
| &#9744;   | <details><summary>Configure server firewalls to only allow access to required ports.</summary> <p> </summary> <p>Using firewalld or IP Tables, configure these firewalls to limit port access to the vault and consul servers.</p> </details> |
| &#9744;   | <details><summary>Disable SSH</summary> <p> </summary> <p>Interaction with Vault is done via the API, even when using the CLI.  As such, there is no reason to have to SSH on to a vault server (if it’s a virtual machine) so SSH should be disabled to mitigate the risk of unauthorised access to the server.</p> </details> |


### **Operational Readiness**

|  |  |
| --------- | ------- |
| &#9744;   | <details><summary>Configure auto-unseal</summary> <p> </summary> <p>Add a seal stanza to the Vault config file to reduce operational burden on operators. For more information check the [auto-unseal documentation here](https://www.vaultproject.io/docs/concepts/seal/#auto-unseal)</p> </details> |
| &#9744;   | <details><summary>PGP encryption of unseal/recovery keys</summary> <p> </summary> <p>Use PGP or Keybase to add an extra layer of security to the distribution of unseal/recovery keys. For more details, see the [official documentation here](https://www.vaultproject.io/docs/concepts/pgp-gpg-keybase/)</p> </details> |
| &#9744;   | <details><summary>Backup / restore practice run</summary> <p> </summary> <p>Practice restoring your Vault platform from a Consul snapshot.  Your backup strategy isn’t complete until you have tested this.</p> </details> |
| &#9744;   | <details><summary>Node rebuild practice run</summary> <p> </summary> <p>Practice building and replacing a node in the vault and consul clusters with zero downtime.</p> </details> |
| &#9744;   | <details><summary>Vault upgrade practice run</summary> <p> </summary> <p>Practice upgrading Vault binaries to newer versions with zero downtime.</p> </details> |
| &#9744;   | <details><summary>Consul upgrade practice run</summary> <p> </summary> <p>Practice upgrading Consul binaries to newer versions with zero downtime.</p> </details> |
| &#9744;   | <details><summary>Load testing</summary> <p> </summary> <p>Consuct load testing to ensure your infrastructure compute resources are sufficient for the load you are expecting. There are projects like [wrk](https://github.com/wg/wrk) That can assist with generating traffic.</p> </details> |
| &#9744;   | <details><summary>Document key holders and contact details</summary> <p> </summary> <p>Ensure unseal/recovery key holders are documented on a Wiki and this document is kept up-to-date</p> </details> |
| &#9744;   | <details><summary>Trusted Broker/Platform pattern</summary> <p> </summary> <p>Choose a platform or broker that your business trusts and use this for secure injection of initial secrets. Examples are using Azure as a trusted platform or using Jenkins as a trusted broker.  Each organisation will differ with regards to what they trust so this should be a business driven decision.</p> </details> |
| &#9744;   | <details><summary>OS Patching strategy</summary> <p> </summary> <p>Document an implement an OS patching strategy, whether it’s updating VM images and replacing VMs with up-to-date images or whether its a controlled direct access update by an operator.</p> </details> |


### **Observability**

|  |  |
| --------- | ------- |
| &#9744;   | <details><summary>Logs shipping to central logs data warehouse</summary> <p> </summary> <p>Logs should be streamed to a central data warehouse as log rotation on the servers should be enabled and logs will be lost locally. A platform like splunk  is ideal for this use case.  There are other viable options available.</p> </details> |
| &#9744;   | <details><summary>Logs data mining scripts written.</summary> <p> </summary> <p>Decide the value that the log data should provide and write some scripts to extract this value from the data. Scripts can be written in python.  Models can also be produced to predict future loads based on existing data sets.  This kind of insight can be useful for planning.</p> </details> |
| &#9744;   | <details><summary>Logs alerting configured</summary> <p> </summary> <p>Some events should generate some kind of alert, for example, a root token being generated should be flagged and alerted on. Ensure these events have alerts configured for them.</p> </details> |

### **Governance and compliance**

|  |  |
| --------- | ------- |
| &#9744;   | <details><summary>Threat Model Exercise </summary> <p> </summary> <p>Conduct a threat modelling exercise using a framework of your organisations choosing and ensure you have documented and mitigated against all identified threats.</p> </details> |
