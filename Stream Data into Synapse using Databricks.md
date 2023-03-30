# Create a streaming data pipeline using Azure Databricks, Event Hub and Synapse


### Requirements

1. Active Azure Subscription
2. Azure Event Hub 
3. Azure Synapse Analytics
4. Azure Storage Account
5. Azure Key Vault (Optional)

### 1.a Create a Event Hub Namespace and Entity

For this project we will be extracting data from the `coincap.io API`. You need to create a unique eventhub namespace, for this project I have created one named `cryptodatastream`. After creating a namespace you can now create an event hub entity, I have created one named `coincaphub`. Setting up these resources is easy you can use this [documentation](https://learn.microsoft.com/en-us/azure/event-hubs/event-hubs-create) to help you get started.

### 1.b Create a Azure Storage account and container. 

You need to create a storage account and a container so that the databricks connection can write data to Azure Synapse. Use these guides to set up a [storage account](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal) and [container](https://learn.microsoft.com/en-us/azure/storage/blobs/blob-containers-portal).

### 1.c Create a Synapse Analytics workspace and Dedicated SQL Pool. 

You also need a Synapse Analytics workspace and a dedicated SQL pool to serve as the data warehouse to store the data processed from the coincap.io API. Use these guides to set up your [workspace](https://learn.microsoft.com/en-us/azure/synapse-analytics/get-started-create-workspace) and [dedicated sql pool]([https://learn.microsoft.com/en-us/azure/storage/blobs/blob-containers-portal](https://learn.microsoft.com/en-us/azure/synapse-analytics/get-started-analyze-sql-pool). Remember to save your credentials somewhere, you will need them later to setup your connections in databricks.

### 1.d Set up Azure Key Vault 

Azure Key Vault is a service provided by Azure that securely stores secrets (private strings like passwords, connection strings etc) and keys. I have used this service in my pipeline, if you do not want to use Key Vault you can simply use the respective secrets directly as strings in databricks. This is link to help setup your [Key Vault](https://medium.com/swlh/a-credential-safe-way-to-connect-and-access-azure-synapse-analytics-in-azure-databricks-1b008839590a). 

### 2. Create the destination table (sink) in the Synapse Dedicated SQL Pool

Run the SQL code below on Azure Synapse Studio, this will create the destination table that will store the data collected and processed from the coincap API.

```	
CREATE TABLE assets.asset_statistics_history_v3
(
    [id_asset_statistics_history] bigint IDENTITY(1,1), --Automatically increases the value for this field for every row insert
    [id] varchar(255),
    [asset_rank] bigint,
    [symbol] varchar(255),
    [asset_name] varchar(255),
    [supply] float,
    [maxSupply] float,
    [marketCapUsd] float,
    [volumeUsd24Hr] float,
    [priceUsd] float,
    [changePercent24Hr] float,
    [vwap24Hr] float,
    [explorer] varchar(255),
    [runtime_timestamp] datetime
)
WITH
(
    DISTRIBUTION = ROUND_ROBIN,
    HEAP
);
```
### 3. Get the connection strings for your services

1. Event Hub: To get the connection string for your event hub entity follow this path `Event Hubs > Event Hub Namespace (What you just created) > Event Hubs (Under Entities) > Event Hub (Entity you created) > Shared access policies`. Click Add, create a name and select Listen. You will need to create another policy for sending data to the event hub, you can use the screenshot below as reference to find your connection string for the respective policy.
We will need the send policy for the python script that will send data to the event hub and the listen policy to read data from databricks

![image](https://user-images.githubusercontent.com/50084105/228878061-116bd708-6155-4a2f-8c20-7d93f68bd4d7.png)


