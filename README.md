# Vault Production Readiness Checklist
This checklist will prepare you launch production-ready vault clusters into any major Cloud provider or on Premise

1. [Storage Backend](#Storage-Backend)
1. [Infrastructure Architecture](#Infrastructure-Architecture])
1. [Monitoring and Alerting](#Monitoring-and-Alerting)
1. [Load Balancing](#Load-Balancing)
1. [Configuration Management](#Configuration-Management)
1. [Operational Management](#Operational-Management)
1. [Compliance and Governance](#Compliance-and-Governance)
1. [Disaster Recovery](#Disaster-Recovery)
1. [Vault Hardening](#Vault-Hardening)

### **Storage Backend**

|  |  |
| --------- | ------- |
| &#9744;   | <details><summary>High Availability</summary> <p> For production Vault clusters, in most cases, businesses will require High Availability for optimum uptime.  If this is the case for your deployment, choose a storage backend that enables High Availability. for more information about each storage Backend option [here.](https://www.vaultproject.io/docs/configuration/storage/index.html) It is a recommended pattern to use [HashiCorp Consul's](https://www.consul.io/) Key/Value store as the storage backend. </p> </details> |
| &#9744;   | <details><summary>Storage Stanza Configuration Secrets</summary> <p> Configuring some storage backend options in the storage stanza requires a connection string which includes a username and password for Vault to connect to them.  These credentials are in plain text and represent a security risk to the storage backend. It's important to use a secure secret injection method to populate the storage stanza in the configuration file.  This enables the configuration file to be placed in source control.  It's also important to add some access control to the configuration file to mitigate against the risk of the file being read on the system. </p> </details> |
| &#9744;   | <details><summary>Gossip Encryption (Consul only)</summary> <p> If using Consul as a storage backend, members of the Consul cluster communicate with each other using a gossip protocol.  This network traffic should be encrypted to minimise security risks.  You can read more about consul encryption [here.](https://www.consul.io/docs/agent/encryption.html)  </p> </details> |
| &#9744;   | <details><summary>ACLs (Consul only)</summary> <p> If using Consul as a storage backend, the path that Vault uses to store it's encrypted data should be protected using Consul's ACL system. Once configured, the Management ACL token should be revoked.  You can read more about configuring Consul's ACL system [here.](https://www.consul.io/docs/acl/index.html)  </p> </details> |
| &#9744;   | <details><summary>Backups</summary> <p> Your chosen Storage Backend is a stateful service and as such, it needs to be backed up on a regular basis.  For consul, the Consul snapshot feature used in conjuction with Disk backups provide a viable implementation of a backup stratgey </p> </details> |


### **Infrastructure Architecture**

|  |  |
| --------- | ------- |
| &#9744;   | <details><summary>Availability Zones</summary> <p> Nodes in Vault clusters (and Consul clusters if being used as a storage backend) should be spread accross two or more failure domains known as Availability zones. The loss of a single Availability zone should not result result in a loss of service. </p> </details> |
| &#9744;   | <details><summary>Machine Images (Virtual Machines only)</summary> <p> If you are deploying your Vault nodes on virtual machines, It is reccomended to build re-usable VM images that can be used to create cluster nodes in an immutable way.  Tools like [Hashicorp Packer](https://packer.io/) are designed to help build repeatable machine images for most virtualised and cloud platform. Machine images should be versioned and should follow a release cycle as new images are produced.</p> </details> |
| &#9744;   | <details><summary>Replication (Enterprise only)</summary> <p> If you are using the Enterprise version of Vault, you can enable replication between two or more Vault clusters in different geographical regions for added protection is Disaster Recovery scenarios.  Replication can be configured in Disaster Recovery Mode or Performance Replication mode.  If you are planing on using Replication, you need to provision infrastructure in an alternative region, with nodes spread accross multiple Availability Zones. For more information about the Enterprise Replication feature, see [the official documentation.](https://www.vaultproject.io/docs/internals/replication/) </p> </details> |
| &#9744;   | <details><summary>Vault accessibility</summary> <p> Vault will likely contain business critical secrets which makes it a prime target for malicious actors. Access to vault to should be restricted to your private networks and not be accessible on the internet.  The Use of Virtual Private Networks is a commonly used approach to allow access to Vault from unknown networks</p> </details> |
| &#9744;   | <details><summary>Compute Resouces</summary> <p> Ensure Hardware servers and Virtual Machines have been appropriately resources in accordance with the [Deployment System Requirements](https://learn.hashicorp.com/vault/operations/ops-reference-architecture#deployment-system-requirements) </p> </details> |


### **Monitoring and Alerting**

|  |  |
| --------- | ------- |
| &#9744;   | <details><summary>Telemetry</summary> <p> </summary> <p> Enable and configure [telemetry](https://www.vaultproject.io/docs/configuration/telemetry/) in the Vault cofiguration file according to the monitoring tools of your choice</p> </details> |
| &#9744;   | <details><summary>Vault Platform Monitoring</summary> <p> Ensure monitoring and Alerting is set-up and configured for Vault platform metrics according to the best practice guidance set out [here.](https://s3-us-west-2.amazonaws.com/hashicorp-education/whitepapers/Vault/Vault-Consul-Monitoring-Guide.pdf) </p> </details> |
| &#9744;   | <details><summary>Consul Platform Monitoring (Consul only)</summary> <p> Ensure monitoring and Alerting is set-up and configured for Consul Platform metrics according to the best practice guidance set out [here.](https://s3-us-west-2.amazonaws.com/hashicorp-education/whitepapers/Vault/Vault-Consul-Monitoring-Guide.pdf) </p> </details> |
| &#9744;   | <details><summary>Infrastructure Monitoring</summary> <p> Ensure monitoring and Alerting is set-up and configured for compute infrastructure according to the best practice guidance set out [here.](https://s3-us-west-2.amazonaws.com/hashicorp-education/whitepapers/Vault/Vault-Consul-Monitoring-Guide.pdf) </p> </details> |

### **Load Balancing**

|  |  |
| --------- | ------- |
| &#9744;   | <details><summary>TLS Encryption</summary> <p> </summary> <p> If your architecture includes a load balancer in front of your vault cluster, Ensure that TLS is enabled and http traffic is re-directed to https</p> </details> |
| &#9744;   | <details><summary>HTTP Redirects</summary> <p> </summary> <p> Ensure that http traffic is re-directed to https</p> </details> |
| &#9744;   | <details><summary>Health Checks</summary> <p> </summary> <p> Ensure health probes are configured on the load balancer. Configure routing rules according to [these response codes](https://www.vaultproject.io/api/system/health.html) </p> </details> |
