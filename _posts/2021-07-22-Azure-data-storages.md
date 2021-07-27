---
hide: false
title: Understand Azure Account storage & Azure SQL (article in progress)
toc: false
comments: true
layout: post
description: An overview of Azure account storage & Azure SQL
categories: [Cloud, Azure, Data Engineering]
---

We are going to talk about two main storage resources found in Azure: the storage account and the Azure SQL. The first provides to the user a unique space to store all kind of data. The second, a kind of SQL Server in the cloud, provides different solutions to deploy SQL relational databases. By going through these two solutions we are going to catch the specific features and options of each service and catch the recommended procedure to deploy them in a secure and scalable way.

## The storage account

A storage account provides a unique namespace in Azure for your data. It supports blobs (including data lake storage Gen2), queue, table store and Azure files. Each object that you store in Azure Storage has an address that includes your **globally unique** storage account name. A combination of the account name and the Azure Storage blob endpoint forms the base address to access objects in your storage account.

![]({{ site.baseurl }}/images/2021-07-27-03-data-storage.png)

Azure storage is a managed service that provides durable (*think redundancy*), secure (*think encryption and access control*), scalable storage. Because it is fully managed, Microsoft handles maintenance and any critical problems. A single Azure subscription can host **up to 200** storage accounts each of them holding up to **500 TB** of data.  Azure storage includes four types of data stores that are accessible via `HTTP` or `HTTPS` using SDKs or REST API:

![]({{ site.baseurl }}/images/2021-07-27-04-data-storage.png)

### Storage account management

#### **Create and access storage account**

You create storage account on the portal or using Azure CLI. There are several parameters to control the configuration of the storage account with the CLI app. See below a few of them:

| Option             | Description                                                  |
| :----------------- | :----------------------------------------------------------- |
| `--name`           | A **storage account name**. Unique across all existing storage account names in Azure. It must be 3 to 24 characters long and can contain only lowercase letters and numbers. |
| `--resource-group` | Use **your_resources_group_name** to place the storage account. |
| `--location`       | Select a location near you (see below). If not specified the storage account will be created in the same location as your resource group. |
| `--sku`            | This decides the storage account performance and replication model. Options include `Premium_LRS`, `Standard_GRS`, `Standard_LRS`, `Standard_RAGRS`, and `Standard_ZRS`. |

To access the services from a storage account you make call through a REST API or use the different SDK available in several programming languages. Once you have your corresponding library / SDK available, you are ready to connect to your storage account. For that you need two things:

- **An Access key**: you have two keys by storage account to rotate them for security
- **A REST API endpoint**: see the table above for the different type of endpoints.

The simplest way to connect with the informations, is to use **storage account connection strings**. A connection string provides all needed connectivity information in a single text string. **BUT BE CAREFUL**, do not put the sensitive information as a plain text. Instead, store connectivity information in a Database, environment variable or configuration file (if `config` file do not track it with your version control service). You can also use *shared access signatures* to give an fine-grained access (time-limited, can restrict permissions).

> For an `.env` file, create it and add the following variable:
>
> ```text
> AZURE_STORAGE_CONNECTION_STRING=<value>
> ```
>
> Then use the Azure CLI do generate the connection string:
>
> ```bash
> az storage account show-connection-string \
>   --resource-group learn-009f04cb-07c8-4c8c-a4d4-17e24253a448 \
>   --query connectionString \
>   --name <name>
> ```
>
> The command should return the value of the connection string. Use it to replace `<value>` from your `.env` file.

#### **Security**

Azure storage provides several tools for security:

**Encryption at rest:** A Storage Service Encryption (SSE) with a 256-AES cipher is used. SSE automatically encrypts data when writing to Azure Storage. Decryption happens automatically when reading data (SSE is included here and not charged for storage). For VMs, you can encrypt the virtual hard disks with Azure Disk Encryption. Keys are stored in Azure key Vault that manages disk encryption keys and secrets.

**Encryption in transit:** One rule here: Always use **HTTPS**.

**Cross-origin resource sharing (CORS):** support CORS uses the HTTP headers so that web application from one domain can access resources from a server at a different domain. With CORS web apps ensure that only authorized content is loaded. CORS is an option you can enable on Storage accounts.

**Role-based access control (RBAC)**: Azure storage supports Azure Active Directory (think kerberos) and RBAC for resources management and data operations to ensure that the client has the right permissions to access the data. RBAC can be scoped at the subscription, resource group, storage account or individual container or queue level.

**Storage account keys:** We have ywo keys by storage account and the possibility to rotate them. *Do not share storage account keys with external third-party applications*.

![]({{ site.baseurl }}/images/2021-07-27-05-data-storage.png)

**Shared access signature (SAS):** As a best practice, use SAS for external third-party clients. A SAS, is a string that contains a security token that can be attached to a URI. Two types of SAS exist:

- *service-level* (read access a specific resource) 
- *account-level* (more permissions like creating files).

**Azure defender for Storage:** It detects unusual and potentially harmful attempts to access or exploit storage accounts. Azure Defender for Storage is currently available for Blob storage, Azure Files, and Azure Data Lake Storage Gen2.

### Azure Blob storage

In Blob storage, every blob lives inside a *blob container*. You can store an unlimited number of blobs in a container and an unlimited number of containers in a storage account. Containers are "flat" — they can only store blobs, not other containers.

Blob storage provides several options for accessing data based on their usage. These options are called *access tier*:

- *Hot access tier* that is optimized for frequent access in the storage account and is the default option
- *Cool access tier* optimized for large amount of data accessed infrequently and **stored at least 30 days**. Accessing data with cool access tier can be more expensive.
- Archive access tier that is only available for block blobs and is optimized for use case where retrieving data may take several hours (high latency). Data needs to be **stored at least 180 days**.

Additionally Blob storage employs *redundancy options* which define where and how data is replicated:

- *Locally redundant storage (LRS)* Data is copied synchronously three times within the primary region.
- *Zone-redundant storage (ZRS)* Data is copied synchronously across three Azure availability zones in the primary region.
- *Geo-redundant storage (GRS)* Data is copied synchronously three times in the primary region, then copied to the secondary region. To access data in the secondary region, the read-access geo-redundant storage (RA-GRS) option must be enabled.
- *Geo-zone-redundant storage (GZRS)* Data is copied synchronously across three Azure availability zones in the primary region, then copied asynchronously to the secondary region. The RA-GRS option must be enabled to access data in the secondary region.

### Azure Data Lake Storage Gen1

Azure Data Lake Storage Gen1 (ADLS Gen1) is an Apache Hadoop file system that is compatible with Hadoop Distributed File System (HDFS) and works with the Hadoop ecosystem. it provides unlimited storage and can store any data in its native format. It uses Azure Active Directory for authentication and access control lists (ACLs) to manage a secure access to the data.

### Azure Data Lake Storage Gen2 (ADLS Gen2)

ADLS Gen2 is dedicated to big data analytics built on top of Azure Blob storage. It's built on by combining both Azure Blob storage and ADLS Gen1 features.  It provides several interesting features as:

- **Hierarchical namespace** with a tree-like organization where objects and files are organized inside directories. Slashes are used in the name to mimic a hierarchical directory structure.
- **Hadoop-compatible access** namely data is stored as if it was HDFS and is accessible directly from Azure Databricks, Azure HDInsight, and Azure Synapse Analytics.
- **Security** with both *access controls* and Portable Operating System Interface (POSIX) permissions. ***You can set permissions at the directory or file level*** for data stored within the data lake.
- **Low cost**
- **Optimized performance** thank to the Azure Blob Filesystem (ABFS) driver. it is is optimized specifically for big data analytics. Moreover, ADSL Gen2 organizes stored data into a hierarchy of directories and subdirectories, much like a file system.
- **Data redundancy** ADLS Gen2 ensure data availability using replication models that provide redundancy locally, or at the zone or region levels.

## Azure SQL

There are three main deployment options and choices that can fit your need if you want to deal with Azure SQL services.

![]({{ site.baseurl }}/images/2021-07-27-06-data-storage.png)

### SQL Server on Azure Virtual Machines

It is a version of SQL server running in a Azure VM. It's basically just SQL Server. This is an Infrastructure as a service (IaaS) with which you need to update and patch both OS and SQL server yourself. If your use case need such deployment, here a few things to consider that we gather in the methodology column in the table below:

![]({{ site.baseurl }}/images/2021-07-27-07-data-storage.png)

### SQL managed instance & SQL database

They are considered as Platform as a Service (PaaS) deployments. These options propose a fully managed engine that automates most of the database management tasks (upgrades, patches, backups and monitoring). Both have a set of common features but have distinct use cases. Notably, use the managed instances if you want to have access to some specific services withing your DB (Machine learning services for example). Additionally, you can specify if you want a single instance or several. 

![]({{ site.baseurl }}/images/2021-07-27-08-data-storage.png)

### Purchasing models, service tiers, and hardware choices

#### Purchasing model

The purchasing model has two options:

- Based on virtual cores (vCore-based)

- Based on database transaction units (DTU-based)

> ***DTU is not available for managed instance***

Microsoft recommends using the **vCore-based** model because it allows to **independently** select compute and storage resources. You pay for compute resources, type and amount of data and log storage, backup storage location (RA-GRS, ZRS or LRS). We focus on the vCore model for the next sections.

#### Service tier

You choose service tier for performance and availability. It is recommended to start with **General Purpose** and adjust later based on your need. The specific features for each service tiers are shown below.

![]({{ site.baseurl }}/images/2021-07-27-09-data-storage.png)

#### Compute tier

Compute tier choices concerns only the vCore-based model with General Purpose service tier. You can choose:

- **Provisioned compute:** for regular usage pattern, fixed amount of resources over time and you are billed regardless of usage. You need to manage the sizing compute resources depending on your workload.
- **Serverless compute:** for intermittent and unpredictable usage. It provides automatic scaling for compute resources and, you are billed only for the amount of compute used. This option also supports automatic pausing (in this setting you only pay for the storage).

### Plan, deploy, and verify Azure SQL

There are several steps to setup your Azure SQL solution. let's dive into some of them

![]({{ site.baseurl }}/images/2021-07-27-10-data-storage.png)

1. **The server**

   For Azure SQL managed instance, giving a server name is the same as in SQL Server. For Databases and elastic pools an Azure SQL Database server is required. This is a logical server that acts as a central administrative point for single or pooled database. It contains logins, firewall rules, auditing rules, threat detections policies and failover groups. For Azure SQL Database server the ***name must be unique across all Azure***

2. **Compute & Storage**

   If you are migrating use a size similar to what you have on-premises. You may use some tools like the Data Migration Assistant SKU to estimate the number of vCores and the Data max size.

3. **Networking**

   Choices for Azure SQL Database or Managed Instance are different:

   - For SQL Databases the default is `No access`. You can then choose a select or private endpoint. You can also add your client IP address if you want to connect from your current client.
   - For Managed Instance you deploy inside an Azure virtual network with a dedicated subnet. As such you have a secure private IP. You can also enable public endpoint if you want to connect through the Internet without VPN. This access is disabled by default

4. **Data collations**

   It is to tell the database engine how to treat certain characters it affects the characteristic of several operations. In SQL server it is defined by OS locale. In Azure SQL managed instance you can set collation upon the creation of the instance (cannot be changed later for the instance but can be changed at the database or column level). With SQL Database you cannot change it at the server level but can alter it at the database level.

5. **Security**

   Opt-in for Azure defender it can detect several threats that we will describe later. Free at the beginning during 15 days.

### Secure your Azure SQL deployment

We are going to see the different layer of security one can leverage for the deployment of Azure SQL solutions. There are divived into 4 main groups: **network security, identity & access, data protection and security management**.

#### Networking security

That is the first security layer you need to engineer and think about. Selective access to your database by services, applications and machines is critical to pave the way towards an secure database architecture. There are different way to tackle this.

- With the `Allow access to Azure services and resources` option set to "yes", you permit to any resources from any region or subscription to access the SQL database. It is a very easy setup to get things up and running and get your SQL database connected to other Azure services as Azure VMs, Azure App Service. However, It is of good practice to not use this approach and prefer to set firewall rules instead of allowing all services to connect to your SQL database.

- With `firewall rules` you add a unique firewall rule for each service or resource to connect. With thses rules you can permit to your on-premises environment to connect to your SQL database. You just need to also add the rules in your on-premise environment.

  ![]({{ site.baseurl }}/images/2021-07-27-12A-data-storage.png)

- Setting the connectivity between resources with firewall rules is tedious. First you will have to enter manually all rules and all IP addresses. Second you might struggle if some IPs are dynamic. Instead you can use virtual network rules that control access for specific networks that contains  machines and / or services. You can allow all connections that come from one specific virtual net- work. This simplifies the challenge of configuring access for all static and dynamic Ips. In Virtual networks your machines expose a private IP address. Beside having private Ips you still connect through a public endpoint.

  ![]({{ site.baseurl }}/images/2021-07-27-12B-data-storage.png)

- In the last scenario for Azure SQL database, we get rid of the public endpoint and make it private: this is the concept of **private link**. You can connect to your Database using a private endpoint. It means that it has a private IP within a specific virtual network. We still have our architecture with Virtual Network, but here rules are unnecessary. Resources that need to connect to the Database must have access to the virtual network where the endpoint is located. As such any connection coming from the Internet will be denied.

- ![]({{ site.baseurl }}/images/2021-07-27-12C-data-storage.png)

- You cannot use Azure Private link with managed instances. For an Azure SQL managed instance, you first need a specific subnet (here the MI-subnet). The subnet is a logical grouping in a virtual network. After the deployment the instances are configured similarly to a private endpoint in a database in Azure SQL Database. With standard networking practice you must enable access to the virtual network where the managed instances are 
  located.

  ![]({{ site.baseurl }}/images/2021-07-27-12D-data-storage.png)

#### Identity

This concerns the use of RBAC and Active Directory (more a topic for Database Engineer). Have a look to the security [best practices](https://docs.microsoft.com/en-us/azure/azure-sql/database/security-best-practice).

For Azure directory there are a couple of rules to remember:

- Use Azure AD managed identity for application that run on Azure virtual machines
- Use Azure AD integrated authentication for for apps running on domain-joined machines outside Azure, assuming the domain has federated with Azure Active Directory.
- Use Azure AD interactive with multifactor authentication for non-Azure machine that are not domain-joined

#### Data encryption

Encrypted connections are forced in Azure SQL Database and Azure SQL Managed Instance. You can optionally specify the inbound Transport Layer Security (TLS). It is of good practice to force encryption on the client to avoid sever negotiation. 

Transparent Data Encryption (TDE) provides encryption for data at rest and is on by default for all new Azure SQL Database instances. For TDE you can use Azure manage your key or bring your own key with the Bring Your Own Key service (BYOK). In the latter scenario you are responsible for the control of the key life-cycle, key usage permissions and auditing operations on keys.

You can also encrypt data at the column level. it is supported just in Azure SQL Database as it is in SQL server. You can also take advantage of the "Always Encrypted" feature. It uses client-side encryption of sensitive data using keys that are never given to the database system. The client encrypts query parameters.

#### Dynamic data masking

You can mask your data so that non-privileged users can't see it but are still able to use query that include the data. For example if you have data with names and e-mail addresses you can mask columns with the following T-SQL commands:

```SQL
ALTER TABLE Data.Membership ALTER COLUMN FirstName
ADD MASKED WITH (FUNCTION = 'partial(1, "xxxxx", 1)')

ALTER TABLE Data.Membership ALTER COLUMN Email
ADD MASKED WITH (FUNCTION = 'email()')

ALTER TABLE Data.Membership ALTER COLUMN DiscountCode 
ADD MASKED WITH (FUNCTION = 'random(1, 100)’)
 
GRANT UNMASK to DataOfficers
```

Data has been masked however people with the fake `DataOfficers` role have access to unmasked data. There are five masking functions:

- Default that produce a full masking according to the data types of the deisgnated fields
- Credit card
- Email
- Random number
- Custom test

![]({{ site.baseurl }}/images/2021-07-27-14-data-storage.png)

#### Data protection wrap-up

To set up and configure data protection, you should:

- Ensure that your apps enforce connection encryption.
- Enable TDE. By default for new databases but you might need to enable it if you migrate your on-premises data.
- Use Dynamic Data masking.
- For advance protection you can configure the "Always Encrypted" feature.

#### Manage security

Once your instance is secured (network, authentication and data), we need to manage security on an ongoing basis. This includes auditing, monitoring, data classification and in the case of Azure SQL, Azure Defender. Here let's give a few words about Azure Defender that is a unified package for advance SQL security capabilities that enables:

- Vulnerability Assessment: a scanning service that provides visibility into security state and suggest approaches to address any potential concerns. You can activate the periodic recurring scans for database checking every seven days. Reports can be sent to admin and a storage account is needed to store the scanning results.
- Advanced Threat Protection: uses advanced monitoring and artificial intelligence to detect whether any of the following threats have occurred: SQL injection, SQL injection vulnerability, Data exfiltration, unsafe action, brute force attempt, anomalous client login.

#### Backup and restore

Azure SQL manages backups and runs if the full recovery model is used. It can restore your database to any point in time. You can even restore a deleted database within the configured retention policy. Most interestingly your backups can be automatically encrypted if TDE is enabled on the logical server or instance. By default, a full database backup is taken once a week. Log backups occur every 5-10 minutes and differential backups every 12-24 hours. Backups files are stored in **Azure storage** in read-access geo-redundant storage (RA-GRS) by default (it is possible to set ZRS or LRS).

The retention period for your data varies between 1 and 35 days. If not enough you can choose long-term retention (LTR). This option automatically creates full backups stored in RA-GRS by default for up to 10 years. LTR is available for Azure SQL Database and in preview for Azure SQL managed instanced.





