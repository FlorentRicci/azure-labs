# Sentiment Analysis - Lab Instructions
The purpose of this lab is to show how you can apply in real-time a machine learning model on streaming data.  This use case will apply sentiment analysis on an incoming stream of Twitter tweets.

## Prerequisites
To execute this lab successfully, you need the following:
* An Azure subscription.  You can create a free one over [here](https://azure.microsoft.com/en-us/free/).
* A Power BI pro subscription.  You can create a 60-day trial over [here](https://signup.microsoft.com/signup?sku=a403ebcc-fae0-4ca2-8c8c-7a907fd6c235&email&ru=https%3A%2F%2Fapp.powerbi.com%3Fpbi_source%3Dweb%26redirectedFromSignup%3D1%26noSignUpCheck%3D1).  
Please remark that you have to use an orgainizational Office365 account.

## Ingest tweets
### Create an Event Hub
First of all, we need a messaging service that can handle huge amounts of streaming data.  Azure Event Hubs is a great service that offers all features to build a realtime data ingestion pipeline.
* Create a new Event Hubs namespace, named _{prefix}-sentiment-analysis-ingestion-eventHubs_
* Provide a unique name, the right subscription, resource group and a location nearby.  Choose _Basic_ as the pricing tier and keep all other settings as default.

![](./images/eventHubs-createNamespace.png "Create Event Hubs namespace")

* Inside the created Event Hubs namespace, create add a new Event Hub, named _{prefix}-sentiment-analysis-ingestion_.  A partition count of 2 and 1 day of message retention is sufficient.  No need to enable the _capture_ feature.

![](./images/eventHubs-createEventHub.png "Create Event Hub")

* Navigate to the previously created Event Hub and add a consumer group with the name _asa_ (referring to Azure Stream Analytics)

![](./images/eventHubs-createConsumerGroup.png "Create consumer group")

* Ensure you are inside the Event Hub tab and click on _Shared access policies_.  Add here a new policy, that gives read access to Azure Stream Analytics.

![](./images/eventHubs-createAccessPolicy.png "Create access policy")

* Click on the created access policy and copy the connection string with primary key.

![](./images/eventHubs-copyConnectionString.png "Copy connection string")

In order to provide a simplified way to ingest tweets, we will use Azure Logic Apps.
* Create a Logic App, named _{prefix}-sentiment-analysis-ingestion-happy_.



