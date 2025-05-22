# Azure cloud shell commands (bash) used for Demos 1-4

## Variables definition

```cli
myresourcegroup="IoTEdgeResourcesDT"
myiothub="myiothubjdf"
location="westeurope"
subscription=$(az account show --query id -o tsv)  # Get current subscription ID
mydevice="myraspi"
```

## Demo 1 - CREATE IOTHUB

```cli
az group create --name $myresourcegroup --location $location
# If necessary for the final assignment you can change the sku [--sku {B1, B2, B3, F1, S1, S2, S3}]
az iot hub create --resource-group $myresourcegroup --name $myiothub --sku S1 --partition-count 2

#create a device identity into the iothub
az iot hub device-identity create --hub-name $myiothub --device-id $mydevice

#get connection string of the device identity
az iot hub device-identity connection-string show --device-id $mydevice --hub-name $myiothub

#monitor the iothub
az iot hub monitor-events --output table --device-id $mydevice --hub-name $myiothub
```


## Demo 2 - SAVE DATA TO A BLOB STORAGE

```cli
mystorage="myiothubdemostorage"
assignee="yesica.diaz@upm.es"

#create a Storage Account 
az storage account create \
--name $mystorage \
--resource-group $myresourcegroup \
--location $location \
--sku Standard_LRS
   
#create a role
az role assignment create \
--assignee $assignee \
--role 'Storage Blob Data Contributor' \
--scope /subscriptions/$subscription/resourceGroups/$myresourcegroup/providers/Microsoft.Storage/storageAccounts/$mystorage

#create a Container in the Storage Account   
az storage account update --name $mystorage --resource-group $myresourcegroup --default-action Allow
az storage container create --account-name $mystorage --name mycontainerforiothub --auth-mode login

#create a custom endpoint
az iot hub message-endpoint create storage-container \
  --hub-name $myiothub \
  --endpoint-name mystorageendpoint \
  --container mycontainerforiothub \
  --connection-string "$(az storage account show-connection-string --name $mystorage --resource-group $myresourcegroup --query connectionString -o tsv)" \
  --resource-group $myresourcegroup \
  --encoding "JSON"


#create a Message Route in IoT Hub
#cold-path
az iot hub message-route create --route-name mystorageroute  --hub-name $myiothub --resource-group $myresourcegroup --source-type DeviceMessages --endpoint-name mystorageendpoint --enabled true --condition 'true'
#hot-path  
az iot hub message-route create --route-name mytelemetryroute  --hub-name $myiothub --resource-group $myresourcegroup --source-type DeviceMessages --endpoint-name events --enabled true --condition 'true' 
```   

   
## Demo 3 - VISUALIZING DATA IN REAL TIME

```cli
mywebapp="mywebappdemoforvisualizingdata"
myconsumergroup="webconsumer"

#create service plan
az appservice plan create --name myserviceplan --resource-group $myresourcegroup --sku FREE
#create web application
az webapp create -n $mywebapp -g $myresourcegroup -p myserviceplan --runtime "NODE:20LTS" 

#add consumer group to your IoT Hub
az iot hub consumer-group create --hub-name $myiothub --name $myconsumergroup

#set IoT Hub Connection String
az webapp config appsettings set --name $mywebapp --resource-group $myresourcegroup --settings IotHubConnectionString="$(az iot hub connection-string show --hub-name $myiothub --policy-name service --output tsv)"

#set Event Hub Consumer Group
az webapp config appsettings set --name $mywebapp --resource-group $myresourcegroup --settings EventHubConsumerGroup=$myconsumergroup

#enable sockets
az webapp config set --name $mywebapp --resource-group $myresourcegroup --web-sockets-enabled true
#enable https
az webapp update --name $mywebapp --resource-group $myresourcegroup --https-only true
```


## Demo 4 - VISUALIZING DATA IN POWERBI

```cli
az login --scope https://management.core.windows.net//.default

mystreamanalytics="mystreamanalyticsdemo"
# create a stream analytics job
az stream-analytics job create \
  --name $mystreamanalytics \
  --resource-group $myresourcegroup \
  --location $location

#add consumer group to your IoT Hub
az iot hub consumer-group create --hub-name $myiothub --name asaconsumer
```

In the azure portal, create an input, an output and define the query as follows:

```cli
SELECT
    *
INTO
    [output]
FROM
    [input]
```
