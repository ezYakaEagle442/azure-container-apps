## **Disclaimer**

**The features described in this workshop might be not yet production-ready, we enable preview-features for the purpose of learning.**


# azure-container-apps
Azure Containers Apps workshop

You can get demos from : 

- [https://docs.microsoft.com/en-us/azure/container-apps/get-started?tabs=bash](https://docs.microsoft.com/en-us/azure/container-apps/get-started?tabs=bash)
- [https://aka.ms/aca-workshop-staging](https://aka.ms/aca-workshop-staging)
- [https://github.com/Azure/reddog-containerapps](https://github.com/Azure/reddog-containerapps)


## ToC

1. Setup [Tools](tools.md)
1. Check [subscription](subscription.md)
1. Setup [environment variables](set-var.md)
1. Setup [pre-requisites](setup-prereq.md)



# Create Azure Container Apps Environment

```sh
az containerapp env create \
  --name $capps_env \
  --resource-group $rg_name \
  --logs-workspace-id $LOG_ANALYTICS_WORKSPACE_CLIENT_ID \
  --logs-workspace-key $LOG_ANALYTICS_WORKSPACE_CLIENT_SECRET \
  --location "$location"
```

# Deploy your first Helllo World

```sh
az containerapp create \
  --name $appName \
  --resource-group $rg_name \
  --environment $capps_env \
  --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest \
  --target-port 80 \
  --ingress 'external' \
  --query configuration.ingress.fqdn

az containerapp show --name $appName -g $rg_name 
```

# Create a revision
```sh
az containerapp revision list --name $appName -g $rg_name -o table

# Create a new revision
az containerapp update --name $appName -g $rg_name --image pinpindock/azconapps:1.1 --verbose
az containerapp revision list --name $appName -g $rg_name -o table
az containerapp revision show --name <REVISION_NAME> --app $appName -g $rg_name

# Deaxctivate first revision. Deactivation stops all running replicas of a revision.
az containerapp revision deactivate --name <REVISION_NAME> --app $appName -g $rg_name
az containerapp revision activate --name <REVISION_NAME> --app $appName -g $rg_name
```

# Monitor the App
```sh
az monitor log-analytics query \
  --workspace $LOG_ANALYTICS_WORKSPACE_CLIENT_ID \
  --analytics-query "ContainerAppConsoleLogs_CL | where ContainerAppName_s == '$appName' | project ContainerAppName_s, Log_s, TimeGenerated | take 3" \
  --out table
```


# Publish revisions with GitHub Actions in Azure Container Apps
```sh
az containerapp github-action add \
  --repo-url "$git_url" \
  --docker-file-path "./Dockerfile" \
  --branch $git_branch \
  --name $appName \
  --resource-group $rg_name \
  --registry-url $acr_registry_url \
  --service-principal-client-id $sp_id \
  --service-principal-client-secret $sp_password \
  --service-principal-tenant-id  $tenantId \
  --token $git_pat

  # --registry-username <REGISTRY_USER_NAME> \
  # --registry-password <REGISTRY_PASSWORD> \

az containerapp github-action show -g $rg_name --name $appName

```


