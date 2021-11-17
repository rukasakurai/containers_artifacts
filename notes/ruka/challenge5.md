
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
```

https://docs.microsoft.com/en-us/azure/aks/csi-secrets-store-identity-access#use-a-user-assigned-managed-identity
```
CLIENT_ID=`az aks show -g $RESOURCE_GROUP -n $CLUSTER_NAME --query identityProfile.kubeletidentity.clientId -o tsv`
echo $CLIENT_ID

77264a2e-19a4-4026-be19-dc20765db7f0

# set policy to access keys in your keyvault
az keyvault set-policy -n $KEYVAULT_NAME --key-permissions get --spn $CLIENT_ID
# set policy to access secrets in your keyvault
az keyvault set-policy -n $KEYVAULT_NAME --secret-permissions get --spn $CLIENT_ID
# set policy to access certs in your keyvault
az keyvault set-policy -n $KEYVAULT_NAME --certificate-permissions get --spn $CLIENT_ID

```

```
az keyvault secret set --vault-name $KEYVAULT_NAME -n dbpassword --value "hL2iz6Kd3"
az keyvault secret set --vault-name $KEYVAULT_NAME -n dbserver --value "sqlserverygo2491.database.windows.net"
az keyvault secret set --vault-name $KEYVAULT_NAME -n dbuser --value "sqladminyGo2491"
```



```
## show secrets held in secrets-store
kubectl exec poi-9478867c-btg7d -n api -- ls /mnt/secrets-store/

## print a test secret 'ExampleSecret' held in secrets-store
kubectl exec poi-9478867c-btg7d -n api -- cat /mnt/secrets-store/dbpassword
kubectl exec poi-9478867c-btg7d -n api -- cat /mnt/secrets-store/dbserver
kubectl exec poi-9478867c-btg7d -n api -- cat /mnt/secrets-store/dbuser
```

# Application Gateway
https://docs.microsoft.com/en-us/azure/application-gateway/ingress-controller-add-health-probes