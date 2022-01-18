# Install Azure-Arc CLI extension

```sh
az extension list-available
# az extension add -n containerapp --yes

az extension add -y --source https://workerappscliextension.blob.core.windows.net/azure-cli-extension/containerapp-0.2.0-py2.py3-none-any.whl 
az extension list -o table
# az extension update --name containerapp
az containerapp --help

```

#  Enable resource providers
```sh
az provider register --namespace Microsoft.Web
az provider show -n Microsoft.Web -o table
# az provider show -n  Microsoft.Web --query  "resourceTypes[?resourceType == 'workerApps']".locations 
az provider show -n  Microsoft.Web --query  "resourceTypes[?resourceType == 'containerApps']".locations
az provider show -n  Microsoft.Web --query  "resourceTypes[?resourceType == 'kubeEnvironments']".locations 
```



# Create RG
```sh
az group create --name $rg_name --location $location
```

# Create Storage

This is not mandatory, you can create a storage account to play with CloudShell

```sh
# https://docs.microsoft.com/en-us/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-create
# https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction#types-of-storage-accounts

az storage account create --name $storage_name --kind StorageV2 --sku Standard_LRS -g $rg_name --location $location --https-only true
az storage account list -g $rg_name

az storage share create --name $fs_share_name --account-name $storage_name
az storage share list --account-name $storage_name
az storage share show --name $fs_share_name --account-name $storage_name

```

# Craete a new Log Analytics workspace
```sh
az monitor log-analytics workspace create --resource-group $rg_name --workspace-name $analytics_workspace_name
LOG_ANALYTICS_WORKSPACE_CLIENT_ID=`az monitor log-analytics workspace show --query customerId -g $rg_name -n $analytics_workspace_name --out tsv`
LOG_ANALYTICS_WORKSPACE_CLIENT_SECRET=`az monitor log-analytics workspace get-shared-keys --query primarySharedKey -g $rg_name -n $analytics_workspace_name --out tsv`
echo "LOG_ANALYTICS_WORKSPACE_CLIENT_ID : " $LOG_ANALYTICS_WORKSPACE_CLIENT_ID
echo "LOG_ANALYTICS_WORKSPACE_CLIENT_SECRET : " $LOG_ANALYTICS_WORKSPACE_CLIENT_SECRET

analytics_workspace_id=$(az monitor log-analytics workspace show --workspace-name $analytics_workspace_name -g $rg_name --query "id" --output tsv)
echo $analytics_workspace_id

```

# Create a revision of your Hello World App
[Fork this sample](https://github.com/ezYakaEagle442/azurecontainerapps-helloworld)
Or just reuse the packaged version of the application available on [Docker Hub](https://hub.docker.com/r/pinpindock/azconapps)

# Create Azure Container Registry

**This registry has to have the Admin User enabled, or the integration with ACA won’t work.**
```sh
az provider register --namespace Microsoft.ContainerRegistry

az acr create --name $acr_registry_name --sku Basic --location $location -g $rg_name --workspace $analytics_workspace_id

acr_registry_id=$(az acr show --name $acr_registry_name --resource-group $rg_name --query "id" --output tsv)
echo "ACR registry ID :" $acr_registry_id

acr_registry_url=$(az acr show --name $acr_registry_name --resource-group $rg_name --query "loginServer" --output tsv)
echo "ACR registry Login server URL :" $acr_registry_url

az acr repository list --name $acr_registry_name
az acr check-health --yes -n $acr_registry_name 

az acr update -n $acr_registry_name --admin-enabled true


sp_password=$(az ad sp create-for-rbac --sdk-auth --name $appName --role contributor --scopes /subscriptions/$subId/resourceGroups/$rg_name --query password --output tsv)
echo $sp_password > $appName-spp.txt
echo "Service Principal Password saved to ./$appName-spp.txt. IMPORTANT Keep your password ..." 

# sp_password=`cat spp.txt`
#sp_id=$(az ad sp show --id $appName --query objectId -o tsv)
#sp_id=$(az ad sp list --all --query "[?appDisplayName=='${appName}'].{appId:appId}" --output tsv)
sp_id=$(az ad sp list --show-mine --query "[?appDisplayName=='${appName}'].{appId:appId}" --output tsv)
echo "Service Principal ID:" $sp_id 
echo $sp_id > $appName-spid.txt
# sp_id=`cat $appName-spid.txt`
az ad sp show --id $sp_id

# az role assignment create --assignee $aks_client_id --role acrpull --scope $acr_registry_id
```

# Create your Personal Access Token

Create your Personal Access Token on [GitHub](https://github.com/settings/tokens)
```sh

```