---
hide: true
title: Azure Data Engineer Notes for Azure Cosmos DB
toc: false
comments: true
layout: post
description: Overview of Cosmos DB
categories: [Cloud, Azure, Data Engineering]
---

## Cosmos DB

To deal with NoSQL / semi-structure data Azure proposes a multi-model database called Cosmos DB.

### Overview of the features

- It is a globally distributed multi-model database serverless service. You don't need to manage the infrastructure or database engine. 
- Cosmos DB provides 99,999% of availability for reads and write.
- The infrastructure has the ability to scale automatically
- It guarantees less than 10-ms latency for reads and indexed and offers replication across several regions

To use Cosmos DB you need to create a Azure Cosmos account. With Cosmos DB we use documents stores, Key/Value store, columnar store or graph store. 

![](/home/dataroots/Documents/BrazLog/images/2021-07-30-01-azure-cosmos-db.png)

Next you need to choose the API depending of the data store you use:

- If you have an existing deployed MongoDB document store, use the MongoDB API
- For a new document store, use the SQL API
- For a new key/value store use the Tale API
- if you have an existing CassandraDB use the Cassandra API
- For a new graph store use the Gremlin API

> Billing:
>
> The metric used to assess the usage of resources and to be charged is based on the request unit (RU) that is a combination of CPU, Memory and IOPS usages. Billing is done on a hourly basis (read 1KB == 1 RU).

### Use the SQL API

The Azure Cosmos DB SQL API allows one to query JSON documents using SQL. It gives you the flexibility of a document store with the familiarity of the SQL language. When creating a database you will go through the following steps:

- Create the Azure Cosmos DB.
- Create a new database (define the Database ID), you can define the number of RUs for the entire database.
- Create a new container (define the Container ID), you can redefine the RUs at the container level.
- Choose your partition key (`/my_partition_key`).
- If you want you can create an analytic store using Azure Synapse Link.

![](/home/dataroots/Documents/BrazLog/images/2021-07-30-02-azure-cosmos-db.png)

Notice here two things:

- The partition key named `/course`
- A lot of metadata is associated with your data. Focus on the `id` property. It can be a self generated value or you can set it yourself. The combination of the partition key and the `id` key helps to identify uniquely the item within the container. The `id` must be a string. you can use the same `id` for different logical partition.

### Partitions in Cosmos DB

Partitioning is used to scale the individual containers in a database to meet the expected performances. Items in the container are divided into subsets called logical partitions. Indeed the items are divided into the various partitions using a partition key.

Let's see the following item:

```json
{
	"participantID": "1",
	"participantName": "Roger",
	"Course": "Analytics"
}
```

Here you could use any property as the partition key. Let's choose the `participantID` for the next following items:

```json
{
	"participantID": "1",{
	"participantID": "3",
	"participantName": "Sandrine",
	"Course": "Analytics"
}
	"participantName": "Roger",
	"Course": "Analytics"
}
{
	"participantID": "2",
	"participantName": "Julien",
	"Course": "Mathematics"
}
{
	"participantID": "3",
	"participantName": "Sandrine",
	"Course": "Analytics"
}
```

Using the `participantID` property as partition key results in 3 logical partitions in the container. For each document you will have one partition. Although it is possible, this might be too much partitions and may cause performance issue when trying to query the data if you scale to more participants.

Let's choose the `Course` as partition key. 

```json
// One parittion
{
	"participantID": "1",
	"participantName": "Roger",
	"Course": "Analytics"
},
{
	"participantID": "3",
	"participantName": "Sandrine",
	"Course": "Analytics"
}

// Second partition
{
	"participantID": "2",
	"participantName": "Julien",
	"Course": "Mathematics"
}
```

We now have just two partitions and we can anticipate that with growing number of participants, we still keep a reasonable number of logical partitions in our container.

#### How to choose the right partition Key

- Choose a property that has a value that does not change a lot.
- But still try to have a reasonable high range of unique values.
- When querying against the data, it is good to include the partition key in your query. Indeed doing that will prompt the engine to filter out the unnecessary partitions. If you do not include the partition key in your query, the engine has to read all the logical partitions. So **choose a partition key that will be frequently used in your query**.
- You can choose the item `id` as partition key as it would improve reads efficiency.
- The only way to have a new partition key is to create a new container. You may have containers with the same data but different partition keys based on the query you want to run against the data.

### Consistency levels in Cosmos DB

Remember that you can enable the replication of your data in Cosmos DB. But, when changes are made to a data located in a specific region, the data can take some time (ms) to replicate across other regions. Let's take the following example:

![](/home/dataroots/Documents/BrazLog/images/2021-07-30-03-azure-cosmos-db.png)

After the write operation one region did not replicate the change yet. So what should happen if a client wants to read the data? What version of the data will it read? The concept of consistency for databases has been frame in the CAP theorem. A fully consistent database means that the client will always read the most recent committed version of the data, to the cost of some latency. Cosmos DB proposes different levels of consistency:

#### Strong

Here the client has to wait before getting the data. It gives highest consistency but lowest throughput (reads).

![](/home/dataroots/Documents/BrazLog/images/2021-07-30-04-azure-cosmos-db.gif)

#### Bounded Staleness

Here the reads can lag behind the writes by at most "n" versions of an item or by "x" time intervals, depending on how you configure it. Outside the staleness windows, the guarantee is similar to what is observed with the strong consistency. But inside the staleness windows guarantee depends on some parameters:

- If clients in the same region for an account with single write region then Strong
- If clients in different regions for an account with a single write region = Consistent prefix
- If clients write to a single region for an account with multiple write regions = Consistent prefix
- If clients write to several regions for an account with multiple write regions = Eventual

![](/home/dataroots/Documents/BrazLog/images/2021-07-30-05-azure-cosmos-db.gif)

#### Session

Here, within a single client session, reads are guaranteed to honor the consistent-prefix, monotonic reads, monotonic writes, read-your-writes,  and write-follows-reads guarantees. This the default and most used consistency model used for Cosmos DB. Outside of the sessions this will depend on the situation:

- If clients in same region for an account with single write region then Consistent Prefix

- if clients in different regions for an account with single write region then Consistent Prefix

- if clients write to a single region for an account with multiple write regions then Consistent Prefix

- if clients write to multiple regions for an account with multiple write regions = Eventual

![](/home/dataroots/Documents/BrazLog/images/2021-07-30-06-azure-cosmos-db.gif)

#### Consistent prefix

In consistent prefix option, updates that are returned contain some  prefix of all the updates, with no gaps. Consistent prefix consistency  level guarantees that reads never see out-of-order writes.

![](/home/dataroots/Documents/BrazLog/images/2021-07-30-07-azure-cosmos-db.gif)

#### Eventual

In eventual consistency, there's no ordering guarantee for reads. In the absence of any further writes, the replicas eventually converge. Eventual consistency is the weakest form of consistency because a client may read the values that are older than the ones it had read before.

![](/home/dataroots/Documents/BrazLog/images/2021-07-30-08-azure-cosmos-db.gif)

### Security aspect with Comos DB

You can set and manage security at three levels:

- **Network:** use the IP firewall rules

- **Authorization:** two master keys (primary and secondary keys, for read/write and read-only keys)

- **Role-based access control:** it provides different levels of access to users defined in Azure AD. Different roles can be assigned:

  - *DocumentDB Account Contributor*, he can manage Azure Cosmos accounts

  - *Cosmos DB Account reader*, he can read the data in an Azure Cosmos DB

  - *Cosmos DB Operator*, he can provision Azure Cosmos accounts, databases and containers but cannot manage the keys to access the data

    > DocumentDB was the previous name for Cosmos DB

**Network security on Azure**:

<img src="/home/dataroots/Documents/BrazLog/images/2021-07-30-09-azure-cosmos-db.png" style="zoom:150%;" />

1. `All networks` or `selected networks`. The first allow traffic from any Azure services. This the option by default. If you want to restrict the connection some some privileged source choose the `selected networks`.
2. With `selected networks` option you can choose to add an existing virtual network or create a new one.
3. Or you can define firewall rules by adding specific IP or IP-ranges to be allowed to connect.

**Authorization Keys**

<img src="/home/dataroots/Documents/BrazLog/images/2021-07-30-10-azure-cosmos-db.png" style="zoom:150%;" />

1. Read-write keys or Read-only keys
2. To connect an application Azure Storage Explorer to your Cosmos DB instance, use the connection string.

**Role-based access control**

<img src="/home/dataroots/Documents/BrazLog/images/2021-07-30-11-azure-cosmos-db.png" style="zoom:150%;" />

1. You can choose the role
2. Assign the role to an Azure AD user, group or service principal
