---
title: Understand Azure Account storage & Azure SQL
toc: false
comments: true
layout: post
description: An overview of Azure account storage & Azure SQL
categories: [Cloud, Azure]
---

##  The storage account

A storage account provides a unique namespace in Azure for your data. It supports blobs (including data lake storage Gen2), queue, table store and Azure files. Every object that you store in Azure Storage has an address that includes your globally unique storage account name. For example, the combination of the account name and the Azure Storage blob endpoint forms the base address for the objects in your storage account.

![]({{ site.baseurl }}/images/2021-07-27-03-data-storage.png)

Azure storage is a manged service that provide durable (think redundancy), secure (think encryption and access control), scalable. Because it is fully managed Microsoft handles maintenance and any critical problems. A single Azure subscription can host up to 200 storage accounts each of them holding up to 500 TB of data.  Azure storage includes four types of data that are accessible via HTTP or HTTPS using SDKs or REST API:

![]({{ site.baseurl }}/images/2021-07-27-04-data-storage.png)

### Storage account management

#### Create and access storage account

You can create storage account on the portal or using de Azure CLI. There are several parameters to control the configuration of the storage account. See below a few of them:

| Option             | Description                                                  |
| :----------------- | :----------------------------------------------------------- |
| `--name`           | A **storage account name**.  Unique across all existing storage account names in Azure. It must be 3 to 24 characters long and can contain only lowercase letters and numbers. |
| `--resource-group` | Use **your_resources_group_name** to place the storage account. |
| `--location`       | Select a location near you (see below). If not specified the storage account will be created in the same location as your resource group. |
| `--sku`            | This decides the storage account performance and replication model. Options include `Premium_LRS`, `Standard_GRS`, `Standard_LRS`, `Standard_RAGRS`, and `Standard_ZRS`. |

To access the services from a storage account you make call through a REST API or use the different SDKs available in several programming languages. Once you have your corresponding library / SDK available you are ready to connect to your storage account. For that you need two things:

- Access key: you have two keys by storage account to rotate them for security
- REST API endpoint: see the figure about storage service and endpoint above.

The simplest way to connect with the informations is to use **storage account connection strings**. A connection string provides all needed connectivity information in a single text string. **BUT BE CAREFUL** do not put the sensitive information as a plain text. Rather store connectivity information in a Database, environment variable or configuration file (if `config` file do not track it with your version control service). You can also use *shared access signatures* to give an fine-grained access (time-limited, can restrict permissions).

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

#### Security 

Azure storage provides several tools for security:

##### Encryption at rest

- Storage Service Encryption (SSE) with a 256-AES cipher. SSE automatically encrypts data when writing to Azure Storage. Deryption happens automatically when reading data (SSE is included here and not charged for storage)
- for VMs, you can encrypt the virtual hard disks with Azure Disk Encryption. Keys are stored in Azure key Vault that manage disk encryption keys and secrets

##### Encryption in transit

- Always use **HTTPS**. 

##### Cross-origin resource sharing (CORS) support

- CORS uses the HTTP headers so that web application from one domain can access resources from a server at a different domain. With CORS web apps ensure that only authorized content is loaded. CORS is an option you can enable on Storage accounts.

##### Role-based access control (RBAC)

- Azure storage supports Azure Active Directory (think kerberos) and RBAC for resources management and data operations to ensure that the client has the right permissions to access the data.
- RBAC can be scoped at the subscription, resource group, storage account or individual container or queue level

##### Storage account keys

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-27-05-data-storage.png)

- Two keys by storage account.
- Possibility to rotate your keys.
- *Do not share storage account keys with external third-party applications*

##### Shared access signature (SAS)

- As a best practice, use SAS for external third-party clients.
- A SAS is a string that contains a security token that can be attached to a URI.
- Two types of SAS: *service-level* (read access a specific resource) & *account-level* (more permissions like creating files)

##### Azure defender for Storage

- It detects unusual and potentially harmful attempts to access or exploit storage accounts
- Azure Defender for Storage is currently available for Blob storage, Azure Files, and Azure Data Lake Storage Gen2.

### Azure Blob storage

In Blob storage, every blob lives inside a *blob container*. You can store an unlimited number of blobs in a container and an unlimited number of containers in a storage account. Containers are "flat" — they can only store blobs, not other containers.

Blob storage provides several options for accessing data based on their usage. These options are called *access tier*:

- *Hot access tier* that is optimized for frequent access in the storage account and is the default option
- *Cool access tier* optimized for large amount of data accessed infrequently and **stored at least 30 days**. Accessing data with cool access tier can be more expensive.
- Archive access tier that is only available for block blobs and is optimized for use case where retrieving data may take several hours (high latency). Data needs to be **stored at least 180 days**.

Additionally Blob storage employs *redundancy options* which define whre and how data is replicated:

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

There are three main deployment options and choices that can fit your need if you want to deal with Azure SQL services: deploy SQL Server on Azure VMs, deploy SQL managed instances or deploy SQL databases.

![]({{ site.baseurl }}/images/2021-07-27-06-data-storage.png)

### SQL Server on Azure Virtual Machines

It is a version of SQL server running in a Azure VM. It's basically just SQL Server. This is an Infrastructure as a service (IaaS) with which you need to update and patch both OS and SQL server yourself.
