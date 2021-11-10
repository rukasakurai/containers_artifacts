
# Key Vault
https://docs.microsoft.com/en-us/azure/aks/csi-secrets-store-driver
```
RESOURCE_GROUP=teamResources
CLUSTER_NAME=team4vnetcluster
KEYVAULT_NAME=team4-kv
LOCATION=japaneast

az aks get-credentials --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP --admin

az aks enable-addons --addons azure-keyvault-secrets-provider --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP

kubectl get pods -n kube-system -l 'app in (secrets-store-csi-driver, secrets-store-provider-azure)'

az keyvault create -n $KEYVAULT_NAME -g $RESOURCE_GROUP -l $LOCATION

az keyvault secret set --vault-name $KEYVAULT_NAME -n ExampleSecret --value MyAKSExampleSecret

## show secrets held in secrets-store
kubectl exec busybox-secrets-store-inline -- ls /mnt/secrets-store/

## print a test secret 'ExampleSecret' held in secrets-store
kubectl exec busybox-secrets-store-inline -- cat /mnt/secrets-store/ExampleSecret
```