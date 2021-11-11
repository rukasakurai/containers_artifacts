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