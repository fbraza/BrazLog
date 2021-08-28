---
hide: true
title: Azure Data Engineer Notes for Stream Analytics
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

Azure Stream Analytics is a real-time analytics and complex event-processing service. It can be used to analyze and process high volumes of streaming data from several sources at the same time. This service is typically used for analysis of real-time telemetry data from IoT enabled devices, web logs and clikcstream analysis, Geospatial analytics. The most commonly adopted method for processing data streams is to analyze new data continuously as it arrives from an event producer, such as Azure Event Hubs. This "live" approach requires more processing power to run computations but offers the ability to gain near-real-time insights. Using a service like Azure Stream Analytics, you can execute calculations and aggregations against arriving data using temporal analysis. The results of those queries can be sent to a Power BI dashboard for real-time visualization and analysis.

You can ingest data from **Azure Event Hubs, Azure IoT and Azure Blob storage**. Azure Event Hubs is a Big Data streaming platform and an event ingestion service that you can use to receive and process millions of events per second. This permits to capture you data in near real-time. The service is fully managed by Microsoft. Interestingly you can also perform queries on your streamed data using the SQL query language allowing to select certain aspects of the input data. Once ingested or processed the data is send to an output stream (Cosmos DB, Azure SQL Database or Azure Synapse SQL for data warehousing, PowerBI). 

Using this service is particularly convenient as no time will be spent in creating an streaming analytics environment. Indeed everything is managed by Azure and provide a 99,9% SLA over tone year. You are billed based on consumption of Streaming Units, so no upfront cost.

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-08-04-01-azure-stream-analytics.png)

When we use Azure Streams analytics we talk about jobs. A defined job is based on one or more data inputs. Each input is a connection to an existing data source. We have two types of input:

- **The Data stream input**: it is an unbounded sequence of events over time. A Azure Stream job **MUST** contain at least one data stream input.
- **The Reference data input**: it is data that slowly changed over time and is stored in Azure Storage to perform correlation and lookups.

You can define output for each job where the data will be send out. Different destination can be chosen as outputs:

- Azure SQL Database.
- Azure Synapse Analytics.
- Azure Cosmos DB.
- Azure Blob storage and Azure Data Lake Gen2.
- PowerBI for vizualization.

## Using Stream Analytics

### Create a stream Analytics Job

1. Sign in to the Azure portal.

2. Select **Create a resource** in the upper left-hand corner of the Azure portal.

3. Select **Analytics** > **Stream Analytics job** from the results list.

4. Fill out the Stream Analytics job page with the following information:

   | **Setting**         | **Suggested value**                               | **Description**                                              |
   | :------------------ | :------------------------------------------------ | :----------------------------------------------------------- |
   | Job name            | MyASAJob                                          | Enter a name to identify your Stream Analytics job. Stream Analytics job name can contain alphanumeric characters, hyphens, and underscores only and it must be between 3 and 63 characters long. |
   | Subscription        | <Your subscription>                               | Select the Azure subscription that you want to use for this job. |
   | Resource group      | asaquickstart-resourcegroup                       | Select the same resource group as your IoT Hub.              |
   | Location            | <Select the region that is closest to your users> | Select geographic location where you can host your Stream Analytics job. Use the location that's closest to your users for better performance and to reduce the data transfer cost. |
   | Streaming units     | 1                                                 | Streaming units represent the computing resources that are required to execute a job. By default, this value is set to 1. To learn about scaling streaming units, refer to [understanding and adjusting streaming units](https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-streaming-unit-consumption) article. |
   | Hosting environment | Cloud                                             | Stream Analytics jobs can be deployed to cloud or edge. Cloud allows you to deploy to Azure Cloud, and Edge allows you to deploy to an IoT Edge device. |

> **Azure Stream Analytics can run in the cloud, for large-scale analytics, or run on IoT Edge or Azure Stack for ultra-low latency analytics.** Azure Stream Analytics uses the same tools and query language on both cloud and the edge

![Azure Stream Analytics can run in the cloud, for large-scale analytics, or run on IoT Edge or Azure Stack for ultra-low latency analytics.](/home/fbraza/Documents/BLOG/BrazLog/images/2021-08-04-02-azure-stream-analytics.png)

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-08-04-03-azure-stream-analytics.png)

## Configure job input

In this section, you will configure an IoT Hub device input to the Stream Analytics job. Use the IoT Hub you created in the previous section of the quickstart.

1. Navigate to your Stream Analytics job.

2. Select **Inputs** > **Add Stream input** > **IoT Hub**.

3. Fill out the **IoT Hub** page with the following values:

   | **Setting**  | **Suggested value** | **Description**                                              |
   | :----------- | :------------------ | :----------------------------------------------------------- |
   | Input alias  | IoTHubInput         | Enter a name to identify the job’s input.                    |
   | Subscription | Your subscription   | Select the Azure subscription that has the storage account you created. The storage account can be in the same or in a different subscription. This example assumes that you have created storage account in the same subscription. |
   | IoT Hub      | MyASAIoTHub         | Enter the name of the IoT Hub you created in the previous section. |

4. Choose carefully the `event serialization format` (JSON, CSV or AVRO) based on you file data.

5. Leave other options to default values and select **Save** to save the settings.

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-08-04-04-azure-stream-analytics.png)

### Configure job output

1. Navigate to the Stream Analytics job that you created earlier.

2. Select **Outputs** > **Add** > **Blob storage**.

3. Fill out the **Blob storage** page with the following values:

   | **Setting**     | **Suggested value**  | **Description**                                              |
   | :-------------- | :------------------- | :----------------------------------------------------------- |
   | Output alias    | BlobOutput           | Enter a name to identify the job’s output.                   |
   | Subscription    | <Your subscription>  | Select the Azure subscription that has the storage account you created. The storage account can be in the same or in a different subscription. This example assumes that you have created storage account in the same subscription. |
   | Storage account | asaquickstartstorage | Choose or enter the name of the storage account. Storage account names are automatically detected if they are created in the same subscription. |
   | Container       | container1           | Select the existing container that you created in your storage account. |

4. Leave other options to default values and select **Save** to save the settings.



## Azure Event Hubs

It is a cloud-based, event-processing service that can receive and process millions of events per second. 

An entity that sends data to the Event Hubs is called a *publisher*. An entity that reads data from the Event Hubs is called a *consumer* or a *subscriber*. Azure Event Hubs sits between these two entities to divide the production (from the publisher) and consumption (to a subscriber) of an event stream. Azure Event Hubs act as a buffer between publisher and consumer.

![An illustration showing an Azure Event Hub placed between four publishers and two subscribers. The Event Hub receives multiple events from the publishers, serializes the events into data streams, and makes the data streams available to subscribers.](https://docs.microsoft.com/en-us/learn/modules/enable-reliable-messaging-for-big-data-apps-using-event-hubs/media/2-event-hub-overview.png)

### Events

An **event** is a small packet of information (a *datagram*) that contains a notification. Events can be published individually, or in batches, but a single publication (individual or batch) can't exceed 1 MB.

### Pricing

There are three pricing tiers for Azure Event Hubs: Basic, Standard, and Dedicated. The tiers differ in terms of supported connections, the number of available Consumer groups, and throughput. When using Azure CLI to create an Event Hubs namespace, if you don't specify a pricing tier, the default of **Standard** (20 Consumer groups, 1000 Brokered connections) is assigned.

## Create and configure new Azure Event Hubs

To create an Event Hub you need to define the Event Hubs **namespace**, then create an Event Hub in that namespace.

### Define an Event Hubs namespace

An Event Hubs namespace is a containing entity for managing one or more Event Hubs. Creating an Event Hubs namespace typically involves the following configuration:

- **Define namespace-level settings**: Define the namespace capacity (configured using **throughput units**), pricing tier, and performance metrics. These settings apply to all the Event Hubs within that namespace. If you don't define these settings, a default value is used: `1` for capacity and `Standard` for pricing tier. Select a **unique name for the namespace**. Access it through this URL: `namespace.servicebus.windows.net`

- You can define the following properties Define the following optional properties:
  - Enable Kafka. This option enables Kafka apps to publish events to the Event Hub.
  - Make this namespace zone redundant. Zone-redundancy replicates data across separate data centers with their independent power, networking, and cooling infrastructures.
  - Enable Auto-Inflate and Auto-Inflate Maximum Throughput Units. **Auto-Inflate provides an automatic scale-up option** by increasing the number of throughput units up to a maximum value. This option is useful to avoid throttling in situations when incoming or outgoing data rates exceed the currently set number of throughput units.

### Azure CLI commands to create an Event Hubs namespace

To create a new Event Hubs namespace, use the `az eventhubs namespace` commands. Here's a brief description of the subcommands you'll use in the exercise.

| Command              | Description                                                  |
| :------------------- | :----------------------------------------------------------- |
| `create`             | Create the Event Hubs namespace.                             |
| `authorization-rule` | All Event Hubs within the same Event Hubs namespace share common connection credentials. You'll need these credentials when you configure apps to send and receive messages using the Event Hub. This command returns the connection string for your Event Hubs namespace. |

### Configure a new Event Hub

After you create the Event Hubs namespace, you can create an Event Hub. When creating a new Event Hub, there are several mandatory parameters.

The following parameters are required to create an Event Hub:

- Event Hub name

  \- Event Hub name that is unique within your subscription and:

  - Is between 1 and 50 characters long.
  - Contains only letters, numbers, periods, hyphens, and underscores.
  - Starts and ends with a letter or number.

- **Partition Count** - The number of partitions required in an Event Hub (between 2 and 32). The partition count should be directly related to the expected number of concurrent consumers and can't be changed after the hub has been created. The partition separates the message stream so that consumer or receiver apps only need to read a specific subset of the data stream. If not defined, this value defaults to *4*.

- **Message Retention** - The number of days (between 1 and 7) that messages will remain available if the data stream needs to be replayed for any reason. If not defined, this value defaults to *7*.

You can also optionally configure an Event Hub to stream data to an Azure Blob storage or Azure Data Lake Store account.

### Azure CLI commands to create an Event Hub

To create a new Event Hub with the Azure CLI, you'll run the `az eventhubs eventhub` command set. Here's a brief description of the subcommands we'll be using.

| Command  | Description                                     |
| :------- | :---------------------------------------------- |
| `create` | Creates the Event Hub in a specified namespace. |
| `show`   | Displays the details of your Event Hub.         |

## Windowing functions

You can perform operations on data contained in temporal windows. All the windowing operations output results at the **end** of the window. 

![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-08-04-07-azure-stream-analytics.png)

Azure Stream analytics has support for five windowing functions:

- **Tumbling window**: here the data stream is segmented into distinct time segments. The windows don't repeat and **DO NOT OVERLAP**. an event cannot belong to more than one tumbling window.

  ![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-08-04-08-azure-stream-analytics.png)

- **Hopping window**: these windows hop forward in time by a fixed period. It may be easy to think of them as Tumbling windows that can overlap.

  ![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-08-04-09-azure-stream-analytics.png)

- **Session window**: this windowing function group events that arrive at similar times, filtering out periods of time where there is no data. It has three main parameters: timeout, maximum duration, and partitioning key (optional).

  ![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-08-04-10-azure-stream-analytics.png)

- **Snapshot windows**: this function groups events that have the same timestamp. Unlike other windowing types, which require a specific window function (such as `SessionWindow()`), you can apply a snapshot window by adding `System.Timestamp()` to the `GROUP BY` clause.

  ![](/home/fbraza/Documents/BLOG/BrazLog/images/2021-08-04-11-azure-stream-analytics.png)
