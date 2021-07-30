---
hide: true
title: Understand Azure Factory and Azure Synapse Analytics
toc: false
comments: true
layout: post
description: An overview of Azure Factory & Azure Synapse Analytics features
categories: [Cloud, Azure, Data Engineering]
---

## Azure Data Factory

As a data engineer you will move data. From source A to sink B and then from source B to sink C and so on. Sometimes you need to move data in a regular basis. You then need to schedule the batch movement of this data. You can have several different data sources and sinks and you need compatible tools to orchestrate the data movement different machine, services and applications. Azure Data Factory (ADF) is the service proposed by azure that can be used for such use cases. It provides a cloud-based ETL & orchestration tool to move but also transform data between different data stores and machines.

### Anatomy of Azure Data Factory

Let's see a very simple example of a orchestration pipeline with ADF. We are going to move a **`CSV`** file from a blob storage to a SQL database. See below a global picture of the key components of Data Factory for this job.

![]({{ site.baseurl }}/images/2021-07-28-01-azure-data-factory.png)

- Data Factory permits to connect you to a large variety of data sources by using an tool called **Linked Service**. They are like connection strings that define the connection information needed to connect to external resources. As such it enables you to ingest data from several sources in order to process them. Additionally, Linked Services can fire up compute resources on demand to ingest / prepare data. For example you can start a HDInsight cluster to prepare data through a hive query. 
- Once the linked service is defined, you can point to the data you want to use thank to **Datasets** objects. A dataset is a named view of data that simply **points or references the data you want to use** in your activities as inputs and outputs. Datasets  identify data within different data stores, such as tables, files,  folders, and documents.
- **Activities** contain the processing and transformation logic of your ETL. Here we have an example with the `Copy Data` activity that is used to ingest from and land data into data stores. Other possibilities are available. Indeed you can use code-free transformations to process and clean the data using the `Mapping Data Flow` activity.  It can execute a stored procedure, Hive Query or push data into a machine learning model to perform analysis. Usually you will have several activities inside you pipeline. You can actually group them into sub-pipelines and decide to schedule them or run them based on specific triggers. in conclusion we distinguish three type of activities: **data movement**, **data transformation** and **control activities**.
- The **integration runtime** provides the compute environment you need for your pipelines. There are three types of Integration runtime: Azure, Self-hosted and Azure-SSIS.

### Azure Data Factory security

To create ADF instances, you need to be a member of the **contributor** or **owner** roles or an **administrator** of the Azure subscription. More specifically to create and manage resources from the Azure portal, you must be in the **Data Factory Contributor** role at the resource group level or above. If you manage resources with PowerShell or the SDK, the **contributor** role at the resource level or above is sufficient. Here some of the permissions you get once you have the contributor role:

- Create, edit, delete data factories and child resources (i.e., datasets, linked services, pipelines, triggers and integaration runtimes).
- Deploy Resources manager templates.
- Manage App insights alerts for Data Factory

### Manage interation runtimes

In Data Factory, an activity defines the action to be performed. A  linked service defines a target data store or a compute service. An  integration runtime provides the infrastructure for the activity and  linked services. It is the compute infrastructure used by ADF. There are three types of integration runtimes:

- Azure
- Self-hosted
- Azure-SSIS

For copy activity you need to be careful with the type of integration runtime:

- To copy data between two **cloud data sources** that both use AZURE IR, ADF will use the Azure runtime.
- To copy data between **a cloud data source and a data source from a private network**, ADF should use the self-hosted.
- For SSIS packages use the Azure SSIS

### Copy activity performance

ADF offers a serverless architecture that permits to develop pipelines to maximize data movement. but these pipelines can consume a lot of network bandwidth and IOPS. There are different ways of measuring and improving the performance of ADF `Copy` activity:

- If you are using the Azure integration runtime check the Data Integration Unit (DIU). It is a measure that represents the power of a single unit in Azure Data Factory. Power is a combination of CPU,  memory, and network resource allocation.
- If you are using the slef hosted integration runtime, try to increase concurrent workload by increasing the number of concurrent jobs by node or add more nodes.
- Using a staged copy of your data can help also but in the case where your pipeline is slowing down a lot check the DIU.

### Data Factory transformation methods

ADF offers a variety of methods to prepare / clean and transform the data. You can choose the one that best fit your needs depending on your projects.

#### Transforming data using Mapping Data Flow

The Mapping Data Flows provide tools for establishing a wide set of data transformation without the need of code. The resulting data flows are executed on an Apache Spark cluster automatically provisionned when you execute the Mapping Data Flow. You have also the Wrangling Data Flow that to perform code free data cleaning and preparation. It allows to do basic data prepation in an interface very similar to excel.

#### Transforming data using compute resources

Azure Data Factory can spin-up compute resources to transform data using specific Big Data / SQL services. For example you can create a pipeline using an analytical data platform such as Spark pools in an Azure Synapse Analytics instance to perform a complex calculation using python. An alternative would be to run the data trasnformation in Azure Databricks using notebooks and code in Scala, Java, R or Python. Below is a table that contain most of the compute resources you can use for your data processing.

| Compute environment                                       | activities                                               |
| --------------------------------------------------------- | -------------------------------------------------------- |
| On-demand HDInsight cluster or your own HDInsight cluster | Hive, Pig, Spark, MapReduce, Hadoop Streaming            |
| Azure Batch                                               | Custom activities                                        |
| Azure Machine Learning Studio Machine                     | Learning activities: Batch Execution and Update Resource |
| Azure Machine Learning                                    | Azure Machine Learning Execute Pipeline                  |
| Azure Data Lake Analytics                                 | Data Lake Analytics U-SQL                                |
| Azure SQL, Azure SQL Data Warehouse, SQL Server           | Stored Procedure                                         |
| Azure Databricks                                          | Notebook, Jar, Python                                    |
| Azure Function                                            | Azure Function activity                                  |

#### Transforming data using SSIS packages

Azure Data Factory provides the ability to lift and shift existing SSIS workload, by creating an Azure-SSIS Integration Runtime to natively  execute SSIS packages.

### Data Factory control flow

In ADF control flow permits to chain activities, branch them altogether based on conditions and define parameters at the pipeline level. It also includes some looping capacity where data is passed to a looping container (ForEach loop for example). Some of the common control flow are quoted below:

- **Chaining activities:** chain activities in a sequence
- **Branching activities:** depends on conditions like `if-condition` activity similar to an `if` statement in programming. When the condition evaluates to `true` a set of activities is executed. If `false` alternative activities are executed.
- **Parameters:** definition of parameters at the pipeline level which allows to pass arugments when invoking the pipeline on-demand or from a trigger. 
- **Looping containers:** these containers defines repetition in a pipeline. You can use `ForEach` activity or the `Until` activity.
- **Trigger based flows:** pipelines can be triggered on-demand (event-based, blob posts) or by clock-time.

### Add parameters to ADF components

#### Parameterize linked services

ADF provides the possibility to parameterize the following linked services:

- Amazon Redshift
- Azure Cosmos DB (SQL API)
- Azure Database for MySQL
- Azure SQL Database
- Azure Synapse Analytics (formerly SQL DW)
- MySQL
- Oracle
- SQL Server
- Generic HTTP
- Generic REST

![]({{ site.baseurl }}/images/2021-07-28-02-azure-data-factory.png)

In the case you are using a connector not present in the list you can instead alter the JSON data that relates to the linked service. In linked service creation/edit blade -> expand "Advanced" at the  bottom -> check "Specify dynamic contents in JSON format" checkbox  -> specify the linked service JSON payload.

![]({{ site.baseurl }}/images/2021-07-28-03-azure-data-factory.png)

#### Global parameters

Setting Global parameters allows you to use constants for consumption in pipeline expressions. A use-case for setting global parameters is when you have multiple pipelines where the parameters names and values are identical.

## Azure Synapse Analytics

Azure Synapse Analytics (ASA) is an integrated analytics platform which combines data warehousing, big data analytics, data integration and visualization into a single environment. You can use ASA to answer the following questions:

- *What happened:* This corresponds to **descriptive analytics** that involves the use of a Data warehouse using the dedicated SQL pool.
- *Why did it happen:* This corresponds to **diagnostic analysis** where you may need to extends your scope of analysis by exploring and finding more data. This can still be done with the SQL pool.
- *What will happen:* This corresponds to **predictivie analysis** based on previous trends or patterns, using the integrated Spark engine with the Azure Synapse Spark pools. These can be used with other services such as Azure Machine learning services or Azure Databricks.
- *What should I do*: This corresponds to **prescriptive analysis** where you need to take actions based on real-time or near-real time analysis of data using predictive analytics. ASA provides this capability by harnessing both Apache Spark, Azure Synapse link and by integrating streaming technologies such as Azure Stream Analytics. Power BI is integrated to the service allowing to build interactive dashboard for real-time analysis.

As such ASA is the right choice if you need to do **modern data warehousing**, **advanced analytics**, **Data exploration and discovery**, **real time analytics** and **data integration** to ingest, prepare, model and serve the data to be used by downstream systems (outside or inside AZA).

![]({{ site.baseurl }}/images/2021-07-28-01-azure-synapase-analytics.png)

### Architecture of Azure Synapse Analytics

![]({{ site.baseurl }}/images/2021-07-28-02-azure-synapse-analytics.png)

We could divide the architecture in 4 main groups:

- **Compute options:** Provisioned SQL Pools (old Azure DW), on-demand SQL Pools (Serverless, can directly query Azure Data Lake, useful for ad-hoc analysis). We also got Provisioned Spark Pools (C#, Scala, Python, R, Java). be careful this is not Databricks.
- **Orchestration options:** Azure Data Factory with ADF Mapping Data Flows
- **Storage options:** Data Lake Store Gen2, a shared Metadata store that permits to share databases and tables between the different engines. You can for example access Spark tables with the SQL engines to create their own objects.
- **Workspace options:** Azure Synapse Studio (a complete development environment to use), Monitoring and Management.

### Provisioned SQL Pools VS on-demand SQL Pools

The dedicated model is referred to as dedicated SQL Pools. It refers to the **data warehousing** features that are generally available in Azure Synapse Analytics. Dedicated SQL pools represent a collection of resources (nodes) that are being provisioned when using Synapse SQL. Use the dedicated setup when you need (i) predictable performance and cost, (ii) create dedicated SQL pools to allocate processing power for **data permanently stored in SQL tables**.

The serverless model is ideal for **unplanned or ad hoc workloads** as performing data exploration or preparing data for data virtualization.

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

### Orchestration

Much of the functionality of Azure Synapse Pipelines come from the Azure Data Factory features and are commonly referred to as Pipelines. You can integrate data pipelines between SQL Pools, Spark Pools and SQL Serverless. For more details have a look to description above about Data Factory.

### Azure Synapse and data warehouse good practices

#### Loading data

Optimizing and speeding up data loads to minimize the impact on the  performance of ongoing queries is a key design goal in data warehousing. Analytical systems are a mixed of loading and querying workloads. Some need data in real-time, other load data periodically. So when you design your strategy for loading data you should ask some of the following questions:

- Where is my data coming from?
- Is the data new? or do you receive changes from existing datasets?
- How often is the data being refreshed, added to or replaced?
- What formats are the data coming in?
- Is the data ingestible as-is? or are transformations and cleansing tasks required?
- Should I transform the data prior to or after loading?
- How complex or robust does the loading process have to be?
- Which takes priority, loading or querying/analysis?

There are different loading methods. You can directly load data with T-SQL and the `COPY` statement, loading data with the pipeline and the `Copy Data` activity or use T-SQL and polybase to define external tables.

Polybase is tool that virtualizes the external data through the SQL pool, so that it can be queried in place like any other table. In Azure Synapse you can use polybase only with data stored in Azure Data Lake. Loading data with polybase implies to follow some steps:

- First set the external data source like Azure Data Lake for example
- Second create an database scoped credential to import data
- Third Create the file format

#### Optimizing query performance

Before talking about performance let's see quickly the architecture of SQL pools.

![]({{ site.baseurl }}/images/2021-07-28-03-azure-synapse-analytics.png)

- Synapse SQL is a distributed computational architecture to process data across several nodes. Compute is uncoupled from storage allowing to scale compute resource independently of the data / storage. For dedicated pool scaling is based on the data warehouse unit (DWU) that is a combination of CPU, memory, and IO whereas scaling is done automatically for serverless SQL pool. 

- Applications connect and issue T-SQL commands to a Control node which is the single point of entry for Synapse SQL. it utilized a distributed processing engine to process data on worker nodes.
