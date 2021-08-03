---
hide: true
title: Azure Data Engineer Notes for Azure Synapse Analytics
toc: false
comments: true
layout: post
description: Overview of Azure Synapse Analytics
categories: [Cloud, Azure, Data Engineering]
---

I am going to share my notes taken during the preparation for the Azure Data Engineer certification (DP-203). I will share them through a series of articles that cover the main concepts and features behind the Azure services needed for Data Engineering. The last set of articles will be focused on cloud architectures and pattern with Azure. If you want to access a specific topic have click to the desirable article below:

- Cosmos DB
- Azure SQL
- Azure Synapse Analytics
- Azure Data Factory
- Azure Databricks
- Azure Event Hub and Azure Stream Analytics

Here, we are going to talk about the Azure Synapse Analytics service which provides a comprehensive approach for ETL / ELT. It allows you to ingest data, prepare and train it (Synapse SQL and Spark pool) and serve it to a Data warehouse. Several connectors exists to then analyze and visualize the data like power BI. We are going to focus mainly and the SQL solutions proposed by Azure Synapse and see their features.

## Azure Synapse Analytics

Azure Synapse Analytics (ASA) is an integrated analytics platform which combines data warehousing, big data analytics, data integration and visualization into a single environment. You can use ASA to answer the following questions:

- *What happened:* This corresponds to **descriptive analytics** that involves the use of a Data warehouse using the dedicated SQL pool.
- *Why did it happen:* This corresponds to **diagnostic analysis** where you may need to extends your scope of analysis by exploring and finding more data. This can still be done with the SQL pool.
- *What will happen:* This corresponds to **predictivie analysis** based on previous trends or patterns, using the integrated Spark engine with the Azure Synapse Spark pools. These can be used with other services such as Azure Machine learning services or Azure Databricks.
- *What should I do*: This corresponds to **prescriptive analysis** where you need to take actions based on real-time or near-real time analysis of data using predictive analytics. ASA provides this capability by harnessing both Apache Spark, Azure Synapse link and by integrating streaming technologies such as Azure Stream Analytics. Power BI is integrated to the service allowing to build interactive dashboard for real-time analysis.

As such ASA is the right choice if you need to do **modern data warehousing**, **advanced analytics**, **Data exploration and discovery**, **real time analytics** and **data integration** to ingest, prepare, model and serve the data to be used by downstream systems (outside or inside AZA).

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-28-01-azure-synapse-analytics.png)

## Architecture of Azure Synapse Analytics

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-28-02-azure-synapse-analytics.png)

We could divide the architecture in 4 main groups:

- **Compute options:** Provisioned SQL Pools (old Azure DW), on-demand SQL Pools (Serverless, can directly query Azure Data Lake, useful for ad-hoc analysis). We also got Provisioned Spark Pools (C#, Scala, Python, R, Java). be careful this is not Databricks.
- **Orchestration options:** Azure Data Factory with ADF Mapping Data Flows
- **Storage options:** Data Lake Store Gen2, a shared Metadata store that permits to share databases and tables between the different engines. You can for example access Spark tables with the SQL engines to create their own objects.
- **Workspace options:** Azure Synapse Studio (a complete development environment to use), Monitoring and Management.

### Synapse SQL Pools

You have two approaches for the Synapses SQL pool: 

- The dedicated model is referred to as dedicated SQL Pools. It refers to the **data warehousing** features that are generally available in Azure Synapse Analytics. Dedicated SQL pools represent a collection of resources (nodes) that are being provisioned when using Synapse SQL. Use the dedicated setup when you need (i) predictable performance and cost, (ii) create dedicated SQL pools to allocate processing power for **data permanently stored in SQL tables**. 
- The serverless model is ideal for **unplanned or ad hoc workloads** as performing data exploration or preparing data for data virtualization.

With Synapse SQL pool you can import Big Data by different ways. You can:

- directly load data with T-SQL and the `COPY` statement

  ```sql
  COPY INTO [dbo].[YOUR_TABLE] FROM 'https://URI_TO_YOUR_BLOB'
  WITH (
  	FIELDTERMINATOR=',',
  	ROWTERMINATOR='0x0A'
  );
  ```

- load data with the pipeline and the `Copy Data` activity

- use **T-SQL and PolyBase** to define external tables. Polybase is tool that virtualizes the external data through the SQL pool, so that it can be queried in place like any other table. **In Azure Synapse you can use Polybase only with data stored in Azure Data Lake**. Loading data with polybase implies to follow some steps (First set the external data source typically Azure Data Lake. Second create an database scoped credential to import data. Third Create the file format.)

#### Architecture of Synapse SQL

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-28-04-azure-synapse-analytics.png)

- Synapse SQL is a distributed computational architecture to process data across several nodes. 
- Compute is uncoupled from storage allowing to scale compute resource independently of the data / storage. For dedicated pool scaling is based on the data warehouse unit (DWU) that is a combination of CPU, memory, and IO whereas scaling is done automatically for serverless SQL pool. 
- Query are sent to the control node. The control node will use the **massively parallel processing engine** (MPP engine) to distribute the query across several worker nodes. The MPP engine optimizes the queries for parallel processing.
- The compute nodes store all of the user data in Azure Storage (Data Lake). More interestingly note the presence of a **Data Movement Service** implemented to optimize the movement of Data across nodes.

#### Usage cost

To monitor usage cost, Azure uses a metric called **Data Warehousing Unit** (DWU) that is a bundle of resources (CPU, memory and IOs). You need to specify the number of DWU you need for the pool. If you want higher performance for your workload at any point in time, just increase the numbers of DWU.

#### Create an Azure Synapse Analytics instance

The procedure is very similar to create an Azure SQL instance. You need to specify the `SQL pool name` and choose a `Server`. If you already have deployed SQL server you can use them. Otherwise create one.

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-28-05-azure-synapse-analytics.png)

### Designing Tables in Azure Synapse Analytics

The concepts described now are some definitions of common Data Warehousing approaches and do not strictly apply to Azure Synapse. Typically in a data warehouse you have three types of tables:

- **Integration tables** that are used to integrate the raw data into the staging area.
- **Fact tables** that contain quantitative data generally loaded from transactional table (e.g, sales).
- **Dimension tables** that contain attribute data that does not change frequently and use to make joins with the fact tables.

When using Azure Synapse Analytics or another data warehousing tool like Hive, you have different choice for the persistence of your tables. In Azure Synapse you have:

- **Regular tables** that are stored in Azure storage as part of the SQL.
- **Temporary tables** that last as long as the session and use local storage for performance.
- **External tables** that are useful to first load data. In Azure, external tables point to and reference data in an Azure Storage blob or Azure Data Lake.

When created tables you need to choose the types of tables you want. be careful choosing the type will have impact on performance:

- **Hash-distributed tables** - The table rows are distributed across the Compute nodes by using a deterministic hash function to assign each row to one **distribution**. A distribution is the basic unit of storage and processing for parallel queries. This particularly good for querying large tables (*more than 2 GB*) or if you have frequent `INSERT, UPDATE, DELETE` operations. 

  > The choice of the distributed columns is important. `NULLABLE` columns are bad candidates as they are going to generate the same hashed result. As such, rows will end up on the same skewed distribution. Similarly, fact tables with columns containing default values or date values are not good candidates for hashed distribution. Large fact tables or historical transaction tables are usually stored as hash-distributed tables. These tables usually have a surrogate key that is monotonically increasing and are used in `JOIN` with other facts or dimension tables. These surrogate keys are good candidates for distributing the data as they are unique.

- **Replicated tables** - A table that is replicated caches a full copy of the table on each compute node. Consequently, replicating a table removes the need to transfer data among compute nodes before a join or aggregation. Replicated tables are best utilized with small tables. Extra storage is required and there is additional overhead that is incurred when writing data, which make large tables impractical.

- **Round-robin tables** - A round-robin distributed table distributes data evenly across the table but without any further optimization. A distribution is first chosen at random and then buffers of rows are assigned to distributions sequentially. **It is recommended to load data into a round-robin table** for staging. `JOIN` operations on round-robin tables require reshuffling data, which takes additional time. 

  > Dimension tables or other lookup tables are stored as round-robin tables. These tables connects to more than one fact tables and optimizing for one `JOIN` may not be the best idea. Usually, dimension tables are smaller which may leave some distributions empty when hash distributed whereas round-robin by definition, guarantees a uniform data distribution.

### Spark pools

Spark pool clusters are groups of computers that are treated as a single computer to handle the execution of commands issued from notebooks. Clusters allow processing of data to be parallelized across many computers to improve scale and performance. It consists of a **Spark Driver** and **Worker nodes**. The Driver node sends work to the Worker nodes and instructs them to pull data from a specified data source. Moreover, you can configure the number of nodes that are required to perform the task.

> Some usage details:
>
> - It takes around two minutes for fewer than 60 nodes and around 5 minutes for more to spin-up. The instance shut down automatically and by-default 5 minutes after the last job has been executed unless it is kept alive by a notebook connection.
> - You can create Spark pool with the portal, Power Shell or C# SDK
> - You can enable auto-scale to let pools automatically scale-up or down based on the workloads. Spark pools can be shut down with no loss of data since data is stored elsewhere on a Data Lake.
> - Spark pools can use Data Lake Store Gen2 and Blob storage.
> - Come with Anaconda librairies pre-installed.

If you need to process big data workloads that cannot be handle by Azure Synapse SQL use the Spark pools. If you have some Spark code already implemented use it! Do not re-implement it with Spark pools.

