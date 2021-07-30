---
hide: true
title: Overview of Azure storage accounts
toc: false
comments: true
layout: post
description: Notes about Azure Storage Accounts
categories: [Cloud, Azure, Data Engineering]
---

We are going to talk about the azure storage accounts which provides to the user a unique space to store all kind of data. We are going to catch the specific features and options of this service and catch the recommended procedure to deploy them in a secure and scalable way.

## The storage account

A storage account provides a unique namespace in Azure for your data. It supports blobs (including data lake storage Gen2), queue, table store and Azure files. Each object that you store in Azure Storage has an address that includes your **globally unique** storage account name. You have distinct storage accounts that proposes different services:

| Type of storage account     | Supported storage services                                   | Redundancy options                         | Usage                                                        |
| --------------------------- | ------------------------------------------------------------ | ------------------------------------------ | ------------------------------------------------------------ |
| Standard general-purpose v2 | Blob (including Data Lake Storage1), Queue, and Table storage, Azure Files | LRS / GRS / RA-GRS / ZRS / GZRS / RA-GZRS2 | Standard storage account type for blobs, file shares, queues, and  tables. Recommended for most scenarios using Azure Storage. Note that if you want support for NFS file shares in Azure Files, use the premium file shares account type. |
| Premium block blobs         | Blob storage (including Data Lake Storage1)                  | LRS  / ZRS2                                | Premium storage account type for block blobs and append blobs. Recommended for scenarios with high transactions rates, or scenarios that use smaller objects or require consistently low storage latency. |
| Premium file shares         | Azure Files                                                  | LRS  / ZRS2                                | Premium storage account type for file shares only. Recommended for enterprise or high-performance scale applications. Use this account type if you want a storage account that supports both SMB and NFS file  shares. |
| Premium page blobs          | Page blobs only                                              | LRS                                        | Premium storage account type for page blobs only.            |

> We will describe more precisely the redundancy options later on

## Storage account services

Because Standard general-purpose v2 has the most features and is the most used account type, we are going to focus on the service it proposes:

![](/home/dataroots/Documents/BrazLog/images/2021-07-27-15-data-storage.png)

 

1. The **Blob** service that is used to store object and large amount of unstructured data on the cloud (videos, images, log files, audio files). It has a flat structure. Everything you store will be store in a container.
2. The **File** service that is used to create file shares on the cloud. The files are accessed via the SMB protocol. You can have folders and files.
3. The **Table** service that is used to store semi-structured NoSQL data in the cloud. It's ideal for storage accessed by web applications and for datasets that don't require complex joins, foreign keys or stored procedures.
4. The **Queue** service that can be used to store large number of messages. It is kind of a middleware because it is delivering cross-communication between applications.

#### Create storage account

**Creation**

![](/home/dataroots/Documents/BrazLog/images/2021-07-27-16-data-storage.png)

1. Your account storage name **must be globally unique**.
2. You can change the **performance** of you account and set them to `Standard` or `Premium`. Setting to premium gives you access to other `Account kinds` with premium services.
3. Note the `Replication` and `Access tier` two very important configuration settings we will talk about later.

**Network connectivity**

![](/home/dataroots/Documents/BrazLog/images/2021-07-27-17-data-storage.png)

1. Here choose carefully your connectivity method. By default it uses a public endpoint available for all networks. You can also select the networks or use a private endpoint and set rules to access it.

**Data protection**

![](/home/dataroots/Documents/BrazLog/images/2021-07-27-18-data-storage.png)

