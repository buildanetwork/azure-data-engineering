# Create a streaming data pipeline using Azure Databricks, Event Hub and Synapse


## Requirements

1. Active Azure Subscription
2. Azure Event Hub 
3. Azure Synapse Analytics
4. Azure Storage Account
5. Azure Databricks
6. Azure Key Vault (Optional)

### 1. Create a Azure Storage account and container. 

You need to create a storage account and a container so that the databricks connection can write data to Azure Synapse. Use these guides to set up a [storage account](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal) and [container](https://learn.microsoft.com/en-us/azure/storage/blobs/blob-containers-portal).

### 2. Create a Synapse Analytics workspace and Dedicated SQL Pool. 

You also need a Synapse Analytics workspace and a dedicated SQL pool to serve as the data warehouse to store the data processed from the coincap.io API. Use these guides to set up your [workspace](https://learn.microsoft.com/en-us/azure/synapse-analytics/get-started-create-workspace) and [dedicated sql pool]([https://learn.microsoft.com/en-us/azure/storage/blobs/blob-containers-portal](https://learn.microsoft.com/en-us/azure/synapse-analytics/get-started-analyze-sql-pool). Remember to save your credentials somewhere, you will need them later to setup your connections in databricks.

### 3. Set up Azure Data Factory

Creating a Azure Data Factory workspace is easy. You can use this [documentation](https://learn.microsoft.com/en-us/azure/data-factory/quickstart-create-data-factory) from Microsoft to setup your factory. 

## Create the destination table (sink) in the Synapse Dedicated SQL Pool

Run the SQL code below on Azure Synapse Studio, this will create the destination table that will store the data collected and processed from the coincap API.

```	
CREATE TABLE [schema].[table name]
(
	[id_asset_price_history] int IDENTITY(1,1),
	[exchangeId] varchar(60),
	[baseId] varchar(60),
	[quoteId] varchar(60),
	[baseSymbol] varchar(60),
	[quoteSymbol] varchar(60),
	[volumeUsd24Hr] float,
	[priceUsd] float,
	[volumePercent] float,
  [runtime_timestamp] datetime
)
WITH
(
  DISTRIBUTION = ROUND_ROBIN,
	HEAP
);
```
## Get the connection strings for your services

### 1. Synapse Analytics Connection String: 
Go to your Synapse Analytics and follow this path to get your connection string `SQL Pools > (Select the SQL Pool you created) > Connection Strings > JDBC Tab > SQL Authentication`. It will look similar to the string below:

`jdbc:sqlserver://{workspacename}.sql.azuresynapse.net:1433;database={dbname};user={userid};password={your_password_here};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.sql.azuresynapse.net;loginTimeout=30;`

### 2. Azure Data Lake Storage Connection String: 
Go to your Storage Account and select `Access Keys` to get your connection string. It will look similar to the string below:

`DefaultEndpointsProtocol=https;AccountName=<storage account>;AccountKey=<key>;EndpointSuffix=core.windows.net`

![Screenshot (21)](https://user-images.githubusercontent.com/50084105/228976604-c7ea7b79-854b-4b3e-8151-cd031e4b0f9a.png)

## Python script to send data to the Event Hub

You can use this code to upload data from the coincap API to Azure Data Lake. This script processes the json received from the API and appends the runtime timestamp after converting epochs to the ISO 8601 format. Note: A JSON array is uploaded to the container.

```
{
   "exchangeId":"Binance",
   "baseId":"bitcoin",
   "quoteId":"tether",
   "baseSymbol":"BTC",
   "quoteSymbol":"USDT",
   "volumeUsd24Hr":"2739949617.7015283309311878",
   "priceUsd":"28170.5081736755863902",
   "volumePercent":"32.5424100913471523",
   "timestamp":"2023-03-30T22:22:23.011000"
}
```

```
import requests
import json
import asyncio
from azure.storage.blob import BlobServiceClient, BlobClient, ContainerClient
import datetime as DT
import os

def get_coin_prices(coin_list):
    payload = ""
    headers = {}
    coin_rates = []
    for coin in coin_list:
        url= "https://api.coincap.io/v2/assets/"+coin+"/markets"
        print(f"\rCoin Data Extracted: {coin_list.index(coin)+ 1}/{len(coin_list)}", end='', flush=True)
        response = requests.request("GET", url, headers=headers, data=payload)
        rates_response = json.loads(response.text)
        ts = rates_response["timestamp"]
        runtime_ts = DT.datetime.utcfromtimestamp(ts/1000).isoformat()
        for rate in rates_response["data"]:
            rate["timestamp"] = runtime_ts
            coin_rates.append(rate)  
    return coin_rates   

def get_asset_data():
    url = "https://api.coincap.io/v2/assets"
    payload={}
    headers = {}
    response = requests.request("GET", url, headers=headers, data=payload)
    asset_data = json.loads(response.text)
    coins = []
    for asset in asset_data["data"]:
        coins.append(asset["id"])
    return coins

coin_list = get_asset_data()
coin_rates_list = get_coin_prices(coin_list)
json_object = json.dumps(coin_rates_list, indent=4)

storage_account_name = "<name>"
storage_account_key = "<key>"
container_name = "<container name>"
directory_name = "<directory>"
connect_str = 'DefaultEndpointsProtocol=https;AccountName=<storage account>;AccountKey=<account key from earlier steps>EndpointSuffix=core.windows.net'

blob_service_client = BlobServiceClient.from_connection_string(connect_str)
container_client = blob_service_client.get_container_client(container_name)
local_path = "./asset-price-data"
file_name =  DT.datetime.now().strftime("%Y-%b-%d-%H-%M") + ".json"

upload_file_path = os.path.join(local_path, file_name)
blob_client = blob_service_client.get_blob_client(container=container_name, blob="asset-exchange-rates/"+file_name)

try:
    with open(upload_file_path, "rb") as data:
            blob_client.upload_blob(data)
            print("\nUploaded"+ file_name)
except:
    print("Issue with Upload")
```
This script should generate the output below



## Running the scripts

1. Enable the trigger in Azure Data Factory. You can do this by going to 'Manage > Triggers'

![image](https://user-images.githubusercontent.com/50084105/229031703-5bd78a6f-1008-42a4-bce9-ba1e927408f4.png)

The trigger will look like that once it has started.
![image](https://user-images.githubusercontent.com/50084105/229030556-fdf0907a-f86f-4984-90ee-c00801d08f36.png)


2. Run the script that uploads the coincap data in a json file to Data Lake Storage. The file was uploaded at 2023-03-30 12:28 UTC.


3. After a few minutes the data will be inserted in the Synapse destination table. You can verify this with the runtime column for the latest entries.

