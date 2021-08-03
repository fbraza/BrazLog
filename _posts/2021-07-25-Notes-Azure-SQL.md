---
hide: true
title: Azure Data Engineer Notes: Storage accounts
toc: false
comments: true
layout: post
description: Overview of Azure SQL
categories: [Cloud, Azure, Data Engineering]
---

I am going to share my notes taken during the preparation for the Azure Data Engineer certification (DP-203). I will share them through a series of articles that cover the main concepts and features behind the Azure services needed for Data Engineering. The last set of articles will be focused on cloud architectures and pattern with Azure. If you want to access a specific topic have click to the desirable article below:

- Cosmos DB
- Azure SQL
- Azure Synapse Analytics
- Azure Data Factory
- Azure Databricks
- Azure Event Hub and Azure Stream Analytics

Here, we are going to talk about the solutions proposed by Azure to deploy and maintain SQL databases. We will focus on understanding  what are the different options proposed and how we can deploy and maintain them according to our needs.

## Azure SQL service

There are three main deployment options and choices that can fit your need if you want to deal with Azure SQL services.

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-27-06-Azure-SQL.png)

### SQL database

This is a fully managed platform as a service (PaaS) that takes care of the database software upgrades,the patches, the backups and the monitoring. using this service you can provide a highly available storage layer for your applications. You have two deployment options: (i) the **single database** which is a managed and isolated database and the (ii) the **elastic pool** which is a collection of single databases that have a shared set of resources (CPU and memory). 

### SQL managed instance

They are considered as Platform as a Service (PaaS) deployments. These options propose a fully managed engine that automates most of the database management tasks (upgrades, patches, backups and monitoring). Both have a set of common features but have distinct use cases. Notably, use the managed instances if you want to have access to some specific services / functions withing your DB (Machine learning services for example). Additionally, you can specify if you want a single instance or a pool of instances. 

### SQL Server on Azure Virtual Machines

It is a version of SQL server running in a Azure VM. It's basically just SQL Server. This is an Infrastructure as a service (IaaS) with which you need to update and patch both OS and SQL server yourself. If your use case need such deployment, here a few things to consider that we gather in the methodology column in the table below:

### Service tiers

#### DTU model

Known as the **D**atabase Transaction **U**nit purchasing model. This model is based on a bundled measure of compute, storage and I/O resources.  There are different service tiers available for the DTU model:

|                              |                      Basic |                   Standard |                    Premium |
| :--------------------------- | -------------------------: | -------------------------: | -------------------------: |
| **Target workload**          | Development and production | Development and production | Development and production |
| **Uptime SLA**               |                     99.99% |                     99.99% |                     99.99% |
| **Maximum backup retention** |                     7 days |                    35 days |                    35 days |
| **CPU**                      |                        Low |          Low, Medium, High |               Medium, High |
| **IOPS (approximate)***      |           1-4 IOPS per DTU |           1-4 IOPS per DTU |           >25 IOPS per DTU |
| **IO latency (approximate)** | 5 ms (read), 10 ms (write) | 5 ms (read), 10 ms (write) |          2 ms (read/write) |
| **Columnstore indexing**     |                        N/A |               S3 and above |                  Supported |
| **In-memory OLTP**           |                        N/A |                        N/A |                  Supported |

Note that the backup retention time is different across the different tiers. However you can still enabled a `Long-Retention` period up to 10 years. In [memory OLTP](https://docs.microsoft.com/en-us/azure/azure-sql/in-memory-oltp-overview#benefits-of-in-memory-technology) is enabled for the `premuim` tier. Premium are chosen for intensive I/O workload.

> Please keep in mind that ***DTU is not available for Azure SQL managed instances***

### Virtual core model

This model allows a lot of flexibility given your use case. You have access to three groups of tiers:

#### General purpose tier

It is a budget-friendly tier designed for most workloads with common performance and availability requirements. Here you can choose:

- **Provisioned compute:** for regular usage pattern, fixed amount of resources over time and you are billed regardless of usage. You need to manage the sizing compute resources depending on your workload.
- **Serverless compute:** for intermittent and unpredictable usage. It provides automatic scaling for compute resources and, you are billed only for the amount of compute used. This option also supports automatic pausing (in this setting you only pay for the storage).

#### The business critical tier

Business critical tier is designed for performance-sensitive workloads with strict availability requirements. This is particularly useful if you want low-latency, high resilience to failures with isolated replicas and use the In-Memory OLTP featue to improve performances.

#### The hypescale tier

It is designed for business workload that need highly scalable storage (+100TB) and read-scale requirements. Cost are between the general purpose and the business critical tiers. This tier is not available for *managed instances*. if you choose this one you cannot go back and change another tier. This is due to the underlying architecture needed to deploy a hyperscale Azure SQL database.

#### Wrap-up

Please see below a table that contains all strucutre the difference between the differnet tiers and display more specific details.

|                -                |       Resource type        |                       General Purpose                        |                          Hyperscale                          |                      Business Critical                       |
| :-----------------------------: | :------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|          **Best for**           |                            | Offers budget oriented balanced compute and storage options. | Most business workloads. Auto-scaling storage size up to 100 TB, fluid vertical and horizontal compute scaling, fast database restore. | OLTP applications with high transaction rate and low IO latency. Offers highest resilience to failures and fast failovers using multiple synchronously updated replicas. |
| **Available in resource type:** |                            |             SQL Database / SQL Managed Instance              |                  Single Azure SQL Database                   |             SQL Database / SQL Managed Instance              |
|        **Compute size**         |        SQL Database        |                        1 to 80 vCores                        |                        1 to 80 vCores                        |                       1 to 128 vCores                        |
|                                 |    SQL Managed Instance    |             4, 8, 16, 24, 32, 40, 64, 80 vCores              |                             N/A                              |             4, 8, 16, 24, 32, 40, 64, 80 vCores              |
|                                 | SQL Managed Instance pools |            2, 4, 8, 16, 24, 32, 40, 64, 80 vCores            |                             N/A                              |                             N/A                              |
|        **Storage type**         |            All             |                        Remote storage                        |             Tiered remote and local SSD storage              |                      Local SSD storage                       |
|        **Database size**        |        SQL Database        |                         1 GB – 4 TB                          |                        40 GB - 100 TB                        |                         1 GB – 4 TB                          |
|                                 |    SQL Managed Instance    |                         32 GB – 8 TB                         |                             N/A                              |                         32 GB – 4 TB                         |
|        **Storage size**         |        SQL Database        |                         1 GB – 4 TB                          |                        40 GB - 100 TB                        |                         1 GB – 4 TB                          |
|                                 |    SQL Managed Instance    |                         32 GB – 8 TB                         |                             N/A                              |                         32 GB – 4 TB                         |
|         **TempDB size**         |        SQL Database        |                       32 GB per vCore                        |                       32 GB per vCore)                       |                       32 GB per vCore                        |
|                                 |    SQL Managed Instance    |                       24 GB per vCore                        |                             N/A                              |             Up to 4 TB - limited by storage size             |
|    **Log write throughput**     |        SQL Database        | **Single databases**: 4.5 MB/s per vCore (max 50 MB/s) **Elastic pools**: 6 MB/s per vCore (max 62.5 MB/s) |                           100 MB/s                           | **Single databases**: 12 MB/s per vCore (max 96 MB/s), **Elastic pools**: 15 MB/s per vCore (max 120 MB/s) |
|                                 |    SQL Managed Instance    |                3 MB/s per vCore (max 22 MB/s)                |                             N/A                              |                4 MB/s per vcore (max 48 MB/s)                |
|        **Availability**         |            All             |                            99.99%                            | 99.95% with one secondary replica, 99.99% with more replicas |     99.99%, 99.995% with zone redundant single database.     |
|           **Backups**           |            All             |            RA-GRS, 1-35 days (7 days by default)             |      RA-GRS, 7 days, fast point-in-time recovery (PITR)      |            RA-GRS, 1-35 days (7 days by default)             |
|       **In-memory OLTP**        |                            |                             N/A                              | Partial support. Memory-optimized table types, table variables, and natively compiled modules are supported. |                          Available                           |
|     **Read-only replicas**      |                            |            0 built-in 0 - 4 using geo-replication            |                        0 - 4 built-in                        |  1 built-in, included in price 0 - 4 using geo-replication   |
|       **Pricing/billing**       |        SQL Database        | vCore, reserved storage, and backup storage are charged. IOPS is not charged. | vCore for each replica and used storage are charged. IOPS not yet charged. | vCore, reserved storage, and backup storage are charged. IOPS is not charged. |
|                                 |    SQL Managed Instance    | vCore, reserved storage, and backup storage is charged. IOPS is not charged |                                                              |                                                              |

### Create, deploy, and verify Azure SQL

For Azure SQL managed instance, giving a server name is the same as in SQL Server. For Databases and elastic pools, an Azure SQL Database server is required. This is a logical server that acts as a central administrative point for single or pooled database. It contains logins, firewall rules, auditing rules, threat detections policies and failover groups. For Azure SQL Database server the ***name must be unique across all Azure***.

With the vCore model we can choose one of the three service-tiers we described before. Here we chose the `General Purpose` tier with the `Provisioned` compute tier. ![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-27-07-Azure-SQL.png)

Next note how the vCore model allows you to specify the amount of resources you want to allocate to your Database Server. The estimated cost is calculated in function of the selected resources.

> Note that you may use some tools like the Data Migration Assistant SKU to estimate the number of vCores and the Data max size if you are not sure about what you should do, especially if you migrate your on-premises databases on Azure Cloud.

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-27-08-Azure-SQL.png)

Next we can choose the connectivity options we want to access the Database. Your choice will depend on your specific requirements. Choices for Azure SQL Database or Managed Instance are different:

- For SQL Databases the default is `No access`. You can then choose a select or private endpoint. You can also add your client IP address if you want to connect from your current client.
- For Managed Instance you deploy inside an Azure virtual network with a dedicated subnet. As such you have a secure private IP. You can also enable public endpoint if you want to connect through the Internet without VPN. This access is disabled by default.

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-27-09-Azure-SQL.png)

Next you can decide which data source you want to use. You can start from scratch, use a sample of data or a Backup. Additionally you also need to specify the **data collations**. It is to tell the database engine how to treat certain characters (it affects the characteristic of several operations).

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-27-10-Azure-SQL.png)

> **Notes about `Database collation`:
>
> - In SQL server it is defined by OS locale. 
> - In Azure SQL managed instance you can set collation upon the creation of the instance (cannot be changed later for the instance but can be changed at the database or column level). 
> - With SQL Database you cannot change it at the server level but can alter it at the database level.

## Working with Elastic pools

Elastic pools are a cost effective solution you can choose when you need to manage several databases. This is particularly useful if you have unpredictable usage demands and don't know the right setting to apply for your databases. Indeed you can group them into a pool. 

> **All the databases on a elastic pool are on a single server and share a set of resources**. 

Similarly to single Azure SQL single Database you can choose between the DTU-based or the vCore-based pricing models. You can also specify the minimum and maximum number of resources shared between the databases that are added to the pool. As such, the database that are not used a lot will not consume much resources in the pool.

## Geo-replication 

This feature allows you to create a readable databases on a server in the same or different region. This particularly useful if you want to ensure business continuity so that an application recovers quickly from some unpredicted disaster and initiate a failover to the secondary database.

> You can have up to four secondaries databases in the same or different regions

Note some specificities:

- The *secondaries* are read-only copies of the database.
- Consequently they are useful to offload / redirect **read-only workload**  on the secondary database.

To set geo-replication go to the `Ge-Replication` section and select a secondary target region. If the targeted region does not have one SQL server already deployed you will need to create one.

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-27-11-Azure-SQL.png)

## Security

#### Network security

That is the first security layer you need to engineer and think about. Selective access to your database by services, applications and machines is critical to pave the way towards an secure database architecture. There are different way to tackle this.

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-27-12A-Azure-SQL.png)

With the `Allow access to Azure services and resources` option set to "yes", you permit to any resources from any region or subscription to access the SQL database. It is a very easy setup to get things up and running and get your SQL database connected to other Azure services as Azure VMs, Azure App Service. However, It is of good practice to not use this approach and prefer to set firewall rules instead of allowing all services to connect to your SQL database. With `firewall rules` you add a unique firewall rule for each service or resource to connect. With thses rules you can permit to your on-premises environment to connect to your SQL database. You just need to also add the rules in your on-premise environment.

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-27-12B-Azure-SQL.png)

Setting the connectivity between resources with firewall rules is tedious. First you will have to enter manually all rules and all IP addresses. Second you might struggle if some IPs are dynamic. Instead you can use virtual network rules that control access for specific networks that contains  machines and / or services. You can allow all connections that come from one specific virtual net- work. This simplifies the challenge of configuring access for all static and dynamic IPs. In Virtual networks your machines expose a private IP address. Beside having private Ips you still connect through a public endpoint.

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-27-12C-Azure-SQL.png)

In the last scenario for Azure SQL database, we get rid of the public endpoint and make it private: this is the concept of **private link**. You can connect to your Database using a private endpoint. It means that it has a private IP within a specific virtual network. We still have our architecture with Virtual Network, but here rules are unnecessary. Resources that need to connect to the Database must have access to the virtual network where the endpoint is located. As such any connection coming from the Internet will be denied.

#### For Azure SQL managed instances

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-27-12D-Azure-SQL.png)

You cannot use Azure Private link with managed instances. For an Azure SQL managed instance, you first need a specific subnet (here the MI-subnet). The subnet is a logical grouping in a virtual network. After the deployment the instances are configured similarly to a private endpoint in a database in Azure SQL Database. With standard networking practice you must enable access to the virtual network where the managed instances are 
located.

### Dynamic Data masking

Dynamic data masking is a technique used to limit the exposure of sensitive data to non-privileged users. You can mask your data so that non-privileged users can't see it but are still able to use query that include the data. For example if you have data with names and e-mail addresses you can mask columns with the following T-SQL commands:

```SQL
ALTER TABLE Data.Membership ALTER COLUMN FirstName
ADD MASKED WITH (FUNCTION = 'partial(1, "xxxxx", 1)')

ALTER TABLE Data.Membership ALTER COLUMN Email
ADD MASKED WITH (FUNCTION = 'email()')

ALTER TABLE Data.Membership ALTER COLUMN DiscountCode 
ADD MASKED WITH (FUNCTION = 'random(1, 100)’)
 
GRANT UNMASK to DataOfficers
```

There are different policies in place to mask the data:

- `Default` - This is a full masking of the data. For numeric data types the value will be set to `0` and for strings,`XXX` characters are used to mask fields.
-  `Credit card` - Here only the last four digits of the credit card are shown.
- `Email` - This exposes the first letter and replaces the domain with `XXX`.com.
- `Random number` - This function generate a random number based on the selected boundaries and the actual data type.
- `Custom text` - Here you can define the exposed prefix, the padding string and the exposed suffix,

Once data masking is et up, you can identify when a user tries to access data from any of the masked column using **Auditing**. Indeed Azure SQL auditing tracks all databases events and writes them to an audit log in your Azure Storage account in **Append Blobs**.

### Data encryption

#### Transparent data encryption (TDE)

It is used to **protect data at rest** for an Azure SQL database. When using TDE, your data, but also backups and transaction logs are encrypted at rest. With the default settings TDE uses a **database encryption key** that is protected by a **built-in server certificate**.

> - You have one distinct built-in certificate by database server
> - You can also use customer-managed keys for the encryption (Bring Your Own Key service - BYOK). Here the encryptions keys can be managed by by the Azure Key vault service. With this scenario you are responsible for the control of the key life-cycle, key usage permissions and auditing operations on keys.
> - To enable TDE for SQL server here the steps yo must follow:
>
>   - Create a master key.
>   - Create or obtain a certificate protected by the master key.
>   - Create a database encryption key and protect it by using the certificate.
>   - Set the database to use encryption.

#### Always encrypted

This is used to protect sensitive data at rest on the server, in-transit between client and server and when the data is in use. As expected, keys are used to encrypt the data and, only client applications or applications servers with access to the keys, will be able to see the data in plain text. You have two types of keys used for the encryption process:

- **Column encryption keys** - It is used to encrypt the data.
- **Column master keys** - It is used to encrypt the column encryption keys. These need to be stored in a trusted location like the Windows Certificate store or the Azure Key Vault service.

`Always encrypted` feature support two types of encryption:

- **Deterministic encryption** - This will ALWAYS generate the same encrypted value for any text value.  You can still perform point lookups, equality joins, grouping and indexing on deterministically encrypted columns.
- **Randomized encryption** - This is more secure because the encrypted value is less predictable, however, you cannot perform, searching, indexing, grouping or joins if columns are encrypted with this method.

To be able to use this feature you need to

- Create an instance of the Azure key Vault service.
- Encrypt columns on SQL server / instance where you can also choose the type of encryption and the key store (Azure key vault)
  - This will create the new master key, the new encryption key and encrypt your column.

## Manage security

Once your instance is secured (network, authentication and data), we need to manage security on an ongoing basis. This includes auditing, monitoring, data classification and in the case of Azure SQL, Azure Defender. Here let's give a few words about Azure Defender that is a unified package for advance SQL security capabilities that enables:

- **Vulnerability Assessment:** a scanning service that provides visibility into security state and suggest approaches to address any potential concerns. You can activate the periodic recurring scans for database checking every seven days. Reports can be sent to an administrator and store in a storage account (Blob store).
- **Advanced Threat Protection:** uses advanced monitoring and artificial intelligence to detect whether any of the following threats have occurred: SQL injection, SQL injection vulnerability, Data exfiltration, unsafe action, brute force attempt, anomalous client login.

> For example if you need to prevent data leakage here what you should do:
>
> - Enable Advanced threat detection
> - Configure the service to send alerts for threat detections of type "Data Exfiltration"
> - Configure the service to send email alerts to the IT

## Monitor and mange your Azure SQL deployment

#### Backup and restore

Azure SQL manages backups and runs if the full recovery model is used. It can restore your database to any point in time. You can even restore a deleted database within the configured retention policy. Most interestingly your backups can be automatically encrypted if TDE is enabled on the logical server or instance. By default, a full database backup is taken once a week. Log backups occur every 5-10 minutes and differential backups every 12-24 hours. Backups files are stored in **Azure storage** in read-access geo-redundant storage (RA-GRS) by default (it is possible to set ZRS or LRS).

The retention period for your data varies between 1 and 35 days. If not enough you can choose long-term retention (LTR). This option automatically creates full backups stored in RA-GRS by default for up to 10 years. LTR is available for Azure SQL Database and in preview for Azure SQL managed instanced.

### Tuning performance in Azure SQL

#### Automatic tuning

Automatic tuning ensures peak performance and stable workloads for Azure SQL Database and Azure SQL Managed Instance   through continuous performance tuning based on AI and machine learning. The automatic tuning options available for both Azure SQL Database and Azure SQL Managed Instance are the following:

| Automatic tuning option                                      | Single database and pooled database support | Instance database support |
| :----------------------------------------------------------- | ------------------------------------------- | ------------------------- |
| **CREATE INDEX** - Identifies indexes that may improve performance of your workload, creates indexes, and automatically verifies that performance of queries has improved. | Yes                                         | No                        |
| **DROP INDEX** - Drops unused (over the last 90 days) and duplicate indexes. Unique indexes,  including indexes supporting primary key and unique constraints, are  never dropped. This option may be automatically disabled when queries  with index hints are present in the workload, or when the workload  performs partition switching. On Premium and Business Critical service  tiers, this option will never drop unused indexes, but will drop  duplicate indexes, if any. | Yes                                         | No                        |
| **FORCE LAST GOOD PLAN**  (automatic plan correction) - Identifies Azure SQL queries using an  execution plan that is slower than the previous good plan, and queries  using the last known good plan instead of the regressed plan. |                                             |                           |

By default the following parameters will be enabled or disabled:

- `FORCE_LAST_GOOD_PLAN = enabled`
- `CREATE_INDEX = disabled`
- `DROP_INDEX = disabled`



### Data Discovery & Classification

