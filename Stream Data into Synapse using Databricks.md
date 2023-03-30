# Create a streaming data pipeline using Azure Databricks, Event Hub and Synapse


### Requirements

1. Active Azure Subscription
2. Azure Event Hub 
3. Azure Synapse Analytics
4. Azure Storage Account

### Creating a Event Hub Namespace and Entity

For this project we will be extracting data from the **coincap.io API**. You need to create a unique eventhub namespace, for this project I have created one named **cryptodatastream**. After creating a namespace you can now create an event hub entity, I have created one named **coincaphub**. Setting up these resources is easy you can use this [documentation](https://learn.microsoft.com/en-us/azure/event-hubs/event-hubs-create) to help you get started.
