---
hide: false
title: Azure Data Engineer Notes for Storage accounts
toc: false
comments: true
layout: post
description: Overview of Azure Storage Accounts
categories: [Cloud, Azure, Data Engineering]
---

I am going to share my notes taken during the preparation for the Azure Data Engineer certification (DP-203). I will share them through a series of articles that cover the main concepts and features behind the Azure services needed for Data Engineering. All articles are treated following a same structure. We describe the Azure services and highlight their specific features, security & monitoring practices. The last set of articles will be focused on cloud architectures and pattern with Azure. If you want to access a specific topic you can choose the topic just below:

- Cosmos DB
- Azure Storage Account
- Azure SQL
- Azure Synapse Analytics
- Azure Data Factory
- Azure Databricks
- Azure Event Hub and Azure Stream Analytics
- Big Data Architectures on Azure

Here, we are going to talk about the azure storage accounts which provides to the user a unique space to store all kind of data. We are going to catch the specific features and options of this service and capture the recommended procedures to deploy them in a secure and scalable way. 

## Definition

A storage account provides a unique namespace in Azure for your data. It supports blobs (including data lake storage Gen2), queue, table store and Azure files. Each object that you store in Azure Storage has an address that includes your **globally unique** storage account name. You have distinct storage accounts that proposes different services:

| Type of storage account     | Supported storage services                                   | Redundancy options                         | Usage                                                        |
| --------------------------- | ------------------------------------------------------------ | ------------------------------------------ | ------------------------------------------------------------ |
| Standard general-purpose v2 | Blob (including Data Lake Storage1), Queue, and Table storage, Azure Files | LRS / GRS / RA-GRS / ZRS / GZRS / RA-GZRS2 | Standard storage account type for blobs, file shares, queues, and  tables. Recommended for most scenarios using Azure Storage. Note that if you want support for NFS file shares in Azure Files, use the premium file shares account type. |
| Premium block blobs         | Blob storage (including Data Lake Storage1)                  | LRS  / ZRS2                                | Premium storage account type for block blobs and append blobs. Recommended for scenarios with high transactions rates, or scenarios that use smaller objects or require consistently low storage latency. |
| Premium file shares         | Azure Files                                                  | LRS  / ZRS2                                | Premium storage account type for file shares only. Recommended for enterprise or high-performance scale applications. Use this account type if you want a storage account that supports both SMB and NFS file  shares. |
| Premium page blobs          | Page blobs only                                              | LRS                                        | Premium storage account type for page blobs only. **Ideal to store VMs disks**. |

## Storage account services

Because Standard general-purpose v2 has the most features and is the most used account type, we are going to focus on the four services it proposes.

![]({{site.baseurl}}/images/2021-08-02-Notes-Azure-storage/fig01.png)

1. The **Blob** service is used to store objects and large amount of unstructured data on the cloud (videos, images, log files, audio files). It has a flat structure. Everything is stored in a container.
2. The **File** service is used to create file shares on the cloud. The files are accessed via the [SMB](https://docs.microsoft.com/en-us/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) protocol. You can have both folders and files.
3. The **Table** service is used to store semi-structured NoSQL data in the cloud. It's ideal for storage accessed by web applications and for datasets that don't require complex joins, foreign keys or stored procedures.
4. The **Queue** service is used to store large number of messages. It works as a [middleware](https://www.ibm.com/cloud/learn/middleware) because it is delivering cross-communication between applications.

## Create storage account

### Creation

Here a set of screenshots that underpin the needed steps to set up a storage account.

![]({{site.baseurl}}/images/2021-08-02-Notes-Azure-storage/fig02.png)

1. Your account storage name **must be globally unique**.
2. You can change the **performance** of you account and set them to `Standard` or `Premium`. Setting to premium gives you access to other `Account kinds` with premium services.
3. Note the `Replication` and `Access tier` two very important configuration settings we will talk about later.

### Network connectivity

![]({{site.baseurl}}/images/2021-08-02-Notes-Azure-storage/fig03.png)

1. Here choose carefully your connectivity method. By default it uses a public endpoint available for all networks. You can also select the networks or use a private endpoint and set rules to access it.

### Data protection

![]({{site.baseurl}}/images/2021-08-02-Notes-Azure-storage/fig04.png)

Once you created your storage account, you can have access to it and use the four storage services as needed.

![]({{site.baseurl}}/images/2021-08-02-Notes-Azure-storage/fig05.png)

**Click on container to create your blob storage.** You will have then to define its name and the connectivity rules to decide if the access is public or private.

# Blob storage security

You have several ways of authorizing the access to your storage account:

-  **First by using the storage account access keys**. You get two keys for your storage account. The keys give access to all services and all data within the storage account. So giving the key is similar to make the new user an administrator. You can rotate keys for security purpose. If you rotate `key1` for `key2`, regenerate the `key1`.

- **Second by using Azure Active Directory.** You can authorized users in Azure AD to work with Azure blob storage. For this you need to assign the required RBAC roles to users who want to use Azure Blob Storage. The default roles are (i) *Storage Blob Data Contributor* (read/write/delete permissions on Blob resources), *Storage Blob Data Reader* (read only access)

- **Third by using shared access signatures**. You can grant secure and temporary access at the level of the storage account without the need of compromising the storage account keys. There you can define which services you want to give access to and what kind of permissions the users will have. See below what it looks like to give permissions with shared keys'

  ![]({{site.baseurl}}/images/2021-08-02-Notes-Azure-storage/fig06.png)

  You can also define access with `Permissions` and `expiry data/time` at the level of your blob object with the help of the shared keys. There you can specify the allowed IP addresses. To sign the shared access signature it uses the storage account key. It return a `Blob SAS Token` or `Blob SAS URL`.

![]({{site.baseurl}}/images/2021-08-02-Notes-Azure-storage/fig07.png)

- **Fourth by using the string connection key** with the AZ CLI or one of the available SDKs

  > For this you need two things:
  >
  > - ***\*An Access key\****: you have two keys by storage account to rotate them for security
  > - ***\*A REST API endpoint\****: see the table above for the different type of endpoints.
  >
  > The simplest way to connect with the informations, is to use ***\*storage account connection strings\****. A connection string provides all needed connectivity information in a single text string. ***\*BUT BE CAREFUL\****, do not put the sensitive information as a plain text. Instead, store connectivity information in a Database, environment variable or configuration file (if `config` file do not track it with your version control service). You can also use **shared access signatures** to give an fine-grained access (time-limited, can restrict permissions).
  >
  > For an `.env` file, create it and add the following variable:
  >
  > ```text
  > AZURE_STORAGE_CONNECTION_STRING=<value>
  > ```
  >
  > Then use the Azure CLI do generate the connection string:
  >
  > ```text
  > az storage account show-connection-string \
  >   --resource-group NAME_OF_YOUR_RESOURCE_GROUP \
  >   --query connectionString \
  >   --name NAME_OF_THE_STORAGE_ACCOUNT
  > ```

# Storage Redundancy

Azure storage always sores multiple copies of the data. There are different options for that:

### Locally-redundant storage (LRS)

- Replicated three times within a single physical location in the primary region.
- 99.999999999% (11 nines) durability of objects over a given year

- This is the lowest-cost redundancy option available.
- The write operation returns successfully only after the data is written to all three replicas 
- *This option does not protect against data center or region wide failures*.

### Zone-redundant storage (ZRS)

- Here the data is replicated three times synchronously across three Azure availability zones in the primary region.
-  99.9999999999% (12 nines) durability of objects over a given year
- *Azure AZs have independent power, cooling and networking, consequently, this option protects against data center / AZs failures.*

### Geo-redundant storage (GRS)

- Here the data is replicated three times within a single physical location in the primary region using LRS.
- 99.99999999999999% (16 nines) durability of objects over a given year
- Then the data is copied asynchronously to a single physical location in the secondary region.
- *This helps to protect against region-wide outages*.
- If the application needs data access in primary and secondary regions you can set the **read-access Geo-redundant storage**.

### Geo-zone-redundant storage

- Here the data is replicated three times synchronously across three Azure availability zones in the primary region using ZRS.
- 99.99999999999999% (16 nines) durability of objects over a given year
- The data is copied asynchronously to a single physical location in the secondary region.
- This helps protect the data against data center failures in the primary region and region-wide outages overall.

- You can set also the **read-access Geo-zone-redundant storage**

![]({{site.baseurl}}/images/2021-08-02-Notes-Azure-storage/fig08.png)

## Blob Storage access tiers

The Blob service in storage accounts provide different access that provide usage and cost flexibility. There three types of access tiers:

- **Hot access tier**: it can be set at the account level and is optimized for data accessed frequently.
- **Cool access tier**: it can be set at the account level and is optimized for data that is infrequently accessed and stored for at least 30 days. Storage cost for this tier is less than for the Hot access tier. However, accessing data (reading, writing) costs more.
- **Archive access tier**: it can be set at the blob level and is optimized for data rarely accessed and stored for at least 180 days. Storage cost is very cheap however access cost is the highest when compared to the *cool* and *hot* tiers. Note that before accessing data in an Archive tier Blob, you first need to *rehydrate* it before can be accessed (this can take several hours).

## Azure Data Lake Storage Gen2

A data lake is **a storage repository that holds a vast amount of raw data in its native format until it is needed**. You can store different types of structured, semi-structure or unstructured data. The Azure Data Lake Storage Gen2 permit to store all kind of data for low costs. `Gen2` is built on top of Azure Blob storage (with all its features) to which has been added additional capabilities coming from the Hadoop-like Azure Data Lake storage Gen1. 

Notably, Azure Data lake Gen2 provides a hierarchical namespace that helps to organize objects/files into a hierarchy of directories (very similar to a is a distributed file system). You can also define `POSIX` permissions of directories and files. Thank to its architecture Azure Data Lake is particularly optimized to store data for Big Data Analytics.

To make your storage account a Data Lake Storage Gen2, you need to enable this option when creating your account.

![]({{site.baseurl}}/images/2021-08-02-Notes-Azure-storage/fig09.png)

