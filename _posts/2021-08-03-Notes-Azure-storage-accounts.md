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

- **First by using the storage account access keys**. You get two keys for your storage account. The keys give access to all services and all data within the storage account. So giving the key is similar to make the new user an administrator. You can rotate keys for security purpose. If you rotate `key1` for `key2`, regenerate the `key1`. If you share the account access key we often talk about a **shared key**.

- >**Second by using shared access signatures** (SAS). You can grant secure and temporary access at the level of the storage account without the need of compromising the storage account keys. There you can define which services you want to give access to and what kind of permissions the users will have. The user does not need to have a identity registered in Azure AD. You can also define access with `Permissions` and `expiry data/time` at the level of your blob object with the help of the shared keys. There you can specify the allowed IP addresses. To sign the shared access signature, it uses the storage account key. It return a `Blob SAS Token` or `Blob SAS URL`.

  >With Shared Key and SAS authorization, Azure RBAC and ACLs have no effect.

- **Third by using RBAC or ACLs.** Both require the user or application to have an identity in Azure AD. RBAC grants coarse-grain access (to the whole storage account). You cannot decide to which blob, files or directory the user will have access but just define wether he will have read and write access over the container. If you want to grant fine-grained access to files and directories use the Azure ACLs.

### RBAC

![]({{site.baseurl}}/images/2021-08-02-Notes-Azure-storage/fig13.png)

- Identities who want to access the storage can be, users or group as administrators in Azure AD. It can also be a created in Azure AD or it could be a **service principal** that is an identity you create for an application. All these Azure AD identities are called **Security Principals**.

- With RBAC we can set pre-defined roles that hold a set of permissions:

  | Role                                                         | Description                                                  |
  | :----------------------------------------------------------- | :----------------------------------------------------------- |
  | [Storage Blob Data Owner](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#storage-blob-data-owner) | Full access to Blob storage containers and data. This access permits the security principal to set the owner an item, and to modify the ACLs of all items. |
  | [Storage Blob Data Contributor](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#storage-blob-data-owner) | Read, write, and delete access to Blob storage containers and blobs. This access does not permit the security principal to set the ownership of an item, but it can modify the ACL of items that are owned by the security principal. |
  | [Storage Blob Data Reader](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#storage-blob-data-reader) | Read and list Blob storage containers and blobs.             |

- With these coarse grain access roles, security principals have access to ALL the data in a Data Lake or ALL the data in a container.

### ACL

ACLs enables you to apply "finer grain" level access to Data Lake Gen 2 directories and files. In other words you can associate a security principal with an access level for files or directories. These associations are captured in an access control list (ACL). Azure AD uses POSIX standards to define the file access.

> RBAC are processed before ACLs. Consequently if you are granting permissions to a security principal using RBAC the ACL will not be applied. 

The level of permissions for files are:

| File        | Directory                                                    |                                                              |
| :---------- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| Read (R)    | Can read the contents of a file                              | Requires **Read** and **Execute** to list the contents of the directory |
| Write (W)   | Can write or append to a file                                | Requires **Write** and **Execute** to create child items in a directory |
| Execute (X) | Does not mean anything in the context of Data Lake Storage Gen2 | Required to traverse the child items of a directory          |

Or using the short forms:

| Numeric form | Short form | What it means          |
| :----------- | :--------- | :--------------------- |
| 7            | `RWX`      | Read + Write + Execute |
| 5            | `R-X`      | Read + Execute         |
| 4            | `R--`      | Read                   |
| 0            | `---`      | No permissions         |

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

## Monitoring the Storage account

To monitor your Azure storage account you can:

- Use Azure Monitor to view information as the amount of storage used
- Use the `Insights` tab in the Azure storage account to access similar metrics
- Use the classic `Diagnostic` settings that are available for Azure storage account and that permit to record and write selected metrics.
- Define some loggin aspects for the storage account.
- **Storage metrics** are recorded in Tables tores. Their names start by `$`.
- **Storage logs** are recorded in a Blob container named `$logs`

### The `Insights` tab

![]({{site.baseurl}}/images/2021-08-02-Notes-Azure-storage/fig10.png)

### The Azure Monitor

![]({{site.baseurl}}/images/2021-08-02-Notes-Azure-storage/fig11.png)

### The classic `Diagnostic` settings

![]({{site.baseurl}}/images/2021-08-02-Notes-Azure-storage/fig12.png)

> Notice the `Logging` part where you can record logs for READ, WRITE and DELETE requests. You can also set a specific retention period

## Conclusion

We tried here to cover the most basic features of the Azure storage account by giving you an overview of the main concepts and services. You can start from here and start digging into the Microsoft documentation that provide much more deep explanation for each services. I am sharing with you some links that might be useful to look at to broaden your knowledge about storage in Azure:

- [Big Data and Azure Data Lake Storage Gen2: Ingest, process, visualize and download data](https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-data-scenarios#download-the-data)
- [Azure Data Lake Storage Gen2: Best practices](https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-best-practices)
