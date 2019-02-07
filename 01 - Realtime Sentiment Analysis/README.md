# Sentiment Analysis - Lab Instructions
The purpose of this lab is to show how you can apply in real-time a machine learning model on streaming data.  This use case will apply sentiment analysis on an incoming stream of Twitter tweets.  The lab focusses on giving a sneak preview on what Azure has to offer, it's not optimized for production usage.

## Prerequisites
To execute this lab successfully, you need the following:
* An Azure subscription.  You can create a free one over [here](https://azure.microsoft.com/en-us/free/).
* An Azure Machine Learning Studio workspace.  you can create a free one over [here](https://studio.azureml.net)
* A Power BI pro subscription.  You can create a 60-day trial over [here](https://signup.microsoft.com/signup?sku=a403ebcc-fae0-4ca2-8c8c-7a907fd6c235&email&ru=https%3A%2F%2Fapp.powerbi.com%3Fpbi_source%3Dweb%26redirectedFromSignup%3D1%26noSignUpCheck%3D1).  
Please remark that you have to use an organizational Office365 account.

## Solution design
The high level solution design of this lab looks like this.

![](./images/lab-solutionDesign.png "Solution design")

* Two Logic Apps are capturing tweets that contain _#happy_ or _#sad_
* These tweets are ingested into Event Hubs
* Azure Stream Analytics performs the sentiment analysis against an Azure Mache Learning web service
* Azure Stream Analytics also calculates the average value over a specific time window
* The results are outputted to Power BI, where they can be easily visualized

## Ingest tweets
### Create an Event Hub
First of all, we need a messaging service that can handle huge amounts of streaming data.  Azure Event Hubs is a great service that offers all features to build a realtime data ingestion pipeline.
* Create a new Event Hubs namespace, named _{prefix}-sentiment-analysis-ingestion-eventHubs_
* Provide a unique name, the right subscription, resource group and a location nearby.  Choose _Basic_ as the pricing tier and keep all other settings as default.

![](./images/eventHubs-createNamespace.png "Create Event Hubs namespace")

* Inside the created Event Hubs namespace, create add a new Event Hub, named _{prefix}-sentiment-analysis-ingestion_.  A partition count of 2 and 1 day of message retention is sufficient.  No need to enable the _capture_ feature.

![](./images/eventHubs-createEventHub.png "Create Event Hub")

### Create a consumer group with an access policy

* Navigate to the previously created Event Hub and add a consumer group with the name _asa_ (referring to Azure Stream Analytics)

![](./images/eventHubs-createConsumerGroup.png "Create consumer group")

* Ensure you are inside the Event Hub tab and click on _Shared access policies_.  Add here a new policy, that gives read access to Azure Stream Analytics.

![](./images/eventHubs-createAccessPolicy.png "Create access policy")

* Click on the created access policy and copy the connection string with primary key.  You'll need this later in this lab.

![](./images/eventHubs-copyConnectionString.png "Copy connection string")

### Create a Logic App
In order to provide a simplified way to ingest tweets, we will use Azure Logic Apps.
* Create a Logic App, named _{prefix}-sentiment-analysis-ingestion-happy_, choose the same region as previously.

![](./images/logicApps-create.png "Create Logic App")

* Open the created Logic App and choose to start from _Blank Logic App_.

![](./images/logicApps-startFromBlank.png "Start from blank Logic App")

### Add a trigger that receives specific tweets

* Add the trigger _When a new tweet is posted_ and authenticate with your Twitter account.  Provide _#happy_ as the hashtag to search for and poll every second.

![](./images/logicApps-addTrigger.png "Add trigger")

### Send the tweets to Event Hubs

* Below the trigger, click on _New step_ to add an action to send to Event Hubs via the _Send event_ action.  Connect the action to the previously created Event Hub namespace.

* Select the correct Event Hub.  Add the Content parameter and provide the following JSON structure:
```json
{
   "text" : "@{triggerBody()['TweetText']}",
   "hashtag" : "#happy",
   "time" : "@{utcNow()}"
}
```

* This should result in the following Logic App:

![](./images/logicApps-final.png "Final Logic App")

* Click _Save_

* Go to the _Overview_ blade and click _Refresh_.  After a while, you should see successful Logic App runs.  All tweets that contain _#happy_ are from now on being ingested into your Event Hub. 

![](./images/logicApps-runs.png "Logic App runs")

> Repeat the above steps to create another Logic App that ingests tweets that contain _#sad_.

## Create a web service that performs sentiment analysis
In this step, we will create an Azure Machine Learning (AML) web service that performs the sentiment analysis.

* Navigate to the Azure AI Gallery [experiment for sentiment analysis].(https://gallery.azure.ai/Experiment/Training-Experiment-for-Twitter-sentiment-analysis-2). 

* Click on _Open in studio_.

![](./images/aml-openInStudio.png "Open in studio")

* Select and/or create a AML Studio workspace.  Click OK if you get a warning about upgrading the experiment to a later version.

![](./images/aml-selectWorkspace.png "Select AML workspace")

* Run the experiment, via the command at the bottom of the page, in order to train the model.  This can take several minutes.

![](./images/aml-runExperiment.png "Run experiment")

* Next, you can click _Deploy web service_.  After a while, you get redirected to the overview page of the created web service.  Copy already the _API key_, as you will need this later in the lab.

![](./images/aml-webServiceOverview.png "Web service overview")

* Via the _Test_ button, you can easily provide a value to be analyzed:

![](./images/aml-webServiceTest.png "Web service test")

* At the bottom of the page, the result appears.

![](./images/aml-webServiceResult.png "Web service result")

* Click now on the _REQUEST/RESPONSE_ link, to go to the _API Help Page_, where you need to copy the web service URL for later usage.

![](./images/aml-webServiceUrl.png "Web service URL")

## Process tweets in realtime

### Create Azure Stream Analytics Job

* Create a Stream Analytics Job, named _{prefix}-sentiment-analysis-asa_.  Give the appropriate resource group and identical location as the previously created services.  Keep _Cloud_ as the hosting environment and set the _Streaming units_ to 1.  The latter will save you some costs.

![](./images/asa-create.png "Create Stream Analytics Job")

### Configure Event Hubs Input

* Let's now create a new _Input_, which should refer to the Event Hub that we created.  Go to the _Inputs_ blade and click _Add stream input_.  Choose _Event Hub_.

* In case you created the Event Hub yourself, you can use the _Select Event Hub from your subscription_ option.  If not, provide the settings manually.  You can retrieve all these settings from the Event Hubs connection string.

![](./images/asa-createInput.png "Create input")

* The best way to verify if the input is configured correctly, is by click on _Sample data_ and specify a timespan of about 20 seconds.  This functionality will connect already to the Event Hub itself, so in case of issues, you'll get an exception.

### Create AML Web Service Function

* To be able to connect to the AML web service, we have to create a new _Function_.  Go to the _Funtions_ blade and click _Add_.  Choose _Azure ML_.  

* Provide the function alias _getSentiment_.  Provide the settings manually by specifying the _Url_ and _API Key_ that you copied previously.

![](./images/asa-createFunction.png "Create function")

### Configure the Power BI output

TODO

### Configure the query

Remove the timestamp BY






















