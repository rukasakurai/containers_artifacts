# Log Analytics
https://docs.microsoft.com/en-us/cli/azure/monitor/log-analytics/workspace?view=azure-cli-latest#az_monitor_log_analytics_workspace_create
```
RESOURCE_GROUP=teamResources
CLUSTER_NAME=team4vnetcluster
KEYVAULT_NAME=team4-kv
LOCATION=japaneast
LA_NAME=team4-log

az monitor log-analytics workspace create --workspace-name $LA_NAME --resource-group $RESOURCE_GROUP --location $LOCATION --retention-time 90
```

https://docs.microsoft.com/en-us/azure/azure-monitor/containers/container-insights-enable-existing-clusters#integrate-with-an-existing-workspace
```
LA_ID=$(az monitor log-analytics workspace show --workspace-name $LA_NAME --resource-group $RESOURCE_GROUP --query id -o tsv)
echo $LA_ID

az aks enable-addons -a monitoring -n $CLUSTER_NAME -g $RESOURCE_GROUP --workspace-resource-id $LA_ID
```

# Create Alert
https://docs.microsoft.com/en-us/azure/azure-monitor/containers/container-insights-log-alerts#create-a-log-query-alert-rule


az monitor alert list -g $RESOURCE_GROUP