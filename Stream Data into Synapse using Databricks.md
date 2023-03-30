# Create a streaming data pipeline using Azure Databricks, Event Hub and Synapse


### Requirements

1. Active Azure Subscription
2. Azure Event Hub 
3. Azure Synapse Analytics
4. Azure Storage Account
5. Azure Key Vault (Optional)

### 1. Create a Event Hub Namespace and Entity

For this project we will be extracting data from the **coincap.io API**. You need to create a unique eventhub namespace, for this project I have created one named **cryptodatastream**. After creating a namespace you can now create an event hub entity, I have created one named **coincaphub**. Setting up these resources is easy you can use this [documentation](https://learn.microsoft.com/en-us/azure/event-hubs/event-hubs-create) to help you get started.

### 2. Create a Azure Storage account and container. 

You need to create a storage account and a container so that the databricks connection can write data to Azure Synapse. Use these guides to set up a [storage account](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal) and [container](https://learn.microsoft.com/en-us/azure/storage/blobs/blob-containers-portal).

### 3. Create a Synapse Analytics workspace and Dedicated SQL Pool. 

You also need a Synapse Analytics workspace and a dedicated SQL pool to serve as the data warehouse to store your processed data from the coincap.io API. Use these guides to set up a [workspace](https://learn.microsoft.com/en-us/azure/synapse-analytics/get-started-create-workspace) and [dedicated sql pool]([https://learn.microsoft.com/en-us/azure/storage/blobs/blob-containers-portal](https://learn.microsoft.com/en-us/azure/synapse-analytics/get-started-analyze-sql-pool). Remember to save your credentials someplace, you will need them later to setup your connections in databricks.
