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
| &#9744;   | <details><summary>High Availability</summary> <p> For production Vault clusters, in most cases, businesses will require High Availability for optimum uptime.  If this is the case for your deployment, choose a storage backend that enables High Availability. for more information about each storage Backend option [here.](https://www.vaultproject.io/docs/configuration/storage/index.html) It is a recommended pattern to use HashiCorp Consul's Key/Value store as the storage backend. </p> </details> |
|  |  |
| &#9744;   | <details><summary>Storage Stanza Configuration Secrets</summary> <p> Configuring some storage backend options in the storage stanza requires a connection string which includes a username and password for Vault to connect to them.  These credentials are in plain text and represent a security risk to the storage backend. It's important to use a secure secret injection method to populate the storage stanza in the configuration file.  This enables the configuration file to be placed in source control.  It's also important to add some access control to the configuration file to mitigate against the risk of the file being read on the system. </p> </details> |