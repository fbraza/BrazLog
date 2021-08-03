---
hide: true
title: Azure Data Engineer Notes: Azure Data Factory
toc: false
comments: true
layout: post
description: Overview of Azure Data Factory
categories: [Cloud, Azure, Data Engineering]
---

I am going to share my notes taken during the preparation for the Azure Data Engineer certification (DP-203). I will share them through a series of articles that cover the main concepts and features behind the Azure services needed for Data Engineering. The last set of articles will be focused on cloud architectures and pattern with Azure. If you want to access a specific topic have click to the desirable article below:

- Cosmos DB
- Azure SQL
- Azure Synapse Analytics
- Azure Data Factory
- Azure Databricks
- Azure Event Hub and Azure Stream Analytics

Here, we are going to talk the cloud-based ETL tool on Azure called, Azure Data Factory. it is a critical service to master if you want to create data scalable and reliable data pipeline in Azure cloud. We are going to browse through the main components and features proposed by Azure Data Factory.

## Anatomy of Azure Data Factory

Azure Data Factory is a cloud-based ETL and data integration service. It allows ingest data for multiple sources and to create data-driven workflow for orchestrating data movement, transformations and processing at scale. Importantly, note that you have the ability to create and schedule data-driven workflows. Let's see a very simple example of a orchestration pipeline with Azure Data Factory. We are going to move a `CSV` file from a blob storage to a SQL database. See below a global picture of the key components of Data Factory for this job.

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-28-01-azure-data-factory.png)

- Data Factory permits to connect you to a large variety of data sources by using an tool called **Linked Service**. They are like connection strings that define the connection information needed to connect to external resources. This helps to connect to external data sources.
- Once the linked service is defined, you can point to the data you want to use thank to **Datasets** objects. A dataset is a named view of data that simply **points or references the data you want to use** in your activities as inputs and outputs. Datasets identify data within different data stores, such as tables, files,  folders, and documents.
- **Activities** contain the processing and transformation logic of your ETL. Here we have an example with the `Copy Data` activity that is used to ingest from and land data into data stores. You can actually group them into sub-pipelines and decide to schedule them or run them based on specific triggers. in conclusion we distinguish three type of activities: **data movement**, **data transformation** and **control activities**. **Pipelines** is a logical grouping of activities.
- The **integration runtime** provides the compute environment you need for your pipelines. There are three types of Integration runtime: Azure, Self-hosted and Azure-SSIS.

## Integration runtimes

The runtime is used to bridge the gap between the activity and the linked service. To copy data for Blob to Azure SQL, you need to have some compute runtime to execute the `COPY` activity. This is what the integration runtime is: **a compute environment for the activity to run**. There are three types of integration runtimes:

- **Azure integration runtime** which helps to execute data flows in Azure by connecting data stores and compute services with **public accessible endpoints**. it is fully managed and serverless compute environment. you don't need to care about patching, scaling or maintenance. it has all required components to move / process data between cloud stores in a secure, performance and reliable manner.

- **Self-hosted integration runtime**: this used to run copy activities between cloud data stores and a data store in a private network. Consequently if you have the data source or destination, consider using this integration runtime. Use self-hosted integration runtime to support data stores that requires bring-your-own driver such as SAP Hana, MySQL, etc. 

  >You can also use the Self-hosted runtime for dispatching the following transform activities against compute resources **in on-premises or Azure Virtual Network:** 
  >
  >- HDInsight Hive activity (BYOC-Bring Your Own Cluster)
  >- HDInsight Pig activity (BYOC)
  >- HDInsight MapReduce activity (BYOC)
  >- HDInsight Spark activity (BYOC)
  >- HDInsight Streaming activity (BYOC)
  >- Azure Machine Learning Studio (classic) Batch Execution activity
  >- Azure Machine Learning Studio (classic) Update Resource activities
  >- Stored Procedure activity
  >- Data Lake Analytics U-SQL activity
  >- Custom activity (runs on Azure Batch)
  >- Lookup activity
  >- Get Metadata activity.
  >
  >Just work for Windows machine as you need to download and install integration runtime on your machine.

- **Azure-SSIS integration runtime** used to lift and shift existing SSIS workloads. you can provision these in either public or private networks.

## Improve Copy activity

Azure Data Factory offers a serverless architecture that permits to develop pipelines to maximize data movement. but these pipelines can consume a lot of network bandwidth and IOPS. There are different ways of measuring and improving the performance of Azure Data Factory `Copy` activity:

- If you are using the Azure integration runtime check the **Data Integration Unit** (DIU). It is a measure that represents the power of a single unit in Azure Data Factory. Power is a combination of CPU,  memory, and network resource allocation.
- If you are using the slef hosted integration runtime, try to increase concurrent workload by increasing the number of concurrent jobs by node or add more nodes.
- Using a staged copy of your data can help also but in the case where your pipeline is slowing down a lot check the DIU.

## Data Factory transformation methods

Azure Data Factory offers a variety of methods to prepare / clean and transform the data. You can choose the one that best fit your needs depending on your projects.

### Transforming data using Mapping Data Flow

The Mapping Data Flows provide tools for establishing a wide set of data transformation without the need of code. In practice Mapping Data Flow help to visually design your data transformations in Azure Data Factory. The flows are then executed as activities within a pipeline using scaled-out Apache Spark clusters. To test your Data lows you can use the built-in debugger.

> You have also the Wrangling Data Flow that to perform code free data cleaning and preparation. It allows to do basic data prepation in an interface very similar to excel.

When creating your Mapping Data Flow instance in Azure Data Factory you first need to specify the source of your data flow (creation of linked service and dataset).

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-28-02-azure-data-factory.png)

Once the source is set up. You can choose between several operation to perform on your data.

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-28-03-azure-data-factory.png)

To save your transformed data, you set a sink destination and map the columns to your destination table (in Azure SQL for example). Once your pipeline is structured and set up you first need to debug it. For that, you need to enable the `Data flow debug` option. It will spin-up a Spark cluster for your debug environment and this may take 5-6 minutes. Once you set the `debug` option everyhting you do will be done in this debug environment. If all the process is errorless you can next publish your pipeline.

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-07-28-04-azure-data-factory.png)

Once published go to the pipeline location, create a pipeline and use the Data Flow you created.

### Transforming data using compute resources

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

### Transforming data using SSIS packages

Azure Data Factory provides the ability to lift and shift existing SSIS workload, by creating an Azure-SSIS Integration Runtime to natively  execute SSIS packages.

## Data Factory control flow

In Azure Data Factory control flow permits to chain activities, branch them altogether based on conditions and define parameters at the pipeline level. It also includes some looping capacity where data is passed to a looping container (`ForEach` loop for example). Some of the common control flow are listed below:

- **Chaining activities:** chain activities in a sequence
- **Branching activities:** depends on conditions like `if-condition` activity similar to an `if` statement in programming. When the condition evaluates to `true` a set of activities is executed. If `false` alternative activities are executed.
- **Parameters:** definition of parameters at the pipeline level which allows to pass arguments when invoking the pipeline on-demand or from a trigger. 
- **Looping containers:** these containers defines repetition in a pipeline. You can use `ForEach` activity or the `Until` activity.
- **Trigger based flows:** pipelines can be triggered on-demand (event-based, blob posts) or by clock-time.

### Triggers

A pipeline can run in response to three triggers signals:

- **Schedule trigger** that invokes the trigger based on a schedule (time).
- **Tumbling window trigger** that invokes the trigger based on periodical interval.
- **Event-based trigger** that invokes trigger based on a specific event (when new data is uploaded to a blob storage).

## Azure Data Factory security

To create Azure Data Factory instances, you need to be a member of the **contributor** or **owner** roles or an **administrator** of the Azure subscription. More specifically to create and manage resources from the Azure portal, you must be in the **Data Factory Contributor** role at the resource group level or above. If you manage resources with PowerShell or the SDK, the **contributor** role at the resource level or above is sufficient. Here some of the permissions you get once you have the contributor role:

- Create, edit, delete data factories and child resources (i.e., datasets, linked services, pipelines, triggers and integaration runtimes).
- Deploy Resources manager templates.
- Manage App insights alerts for Data Factory
