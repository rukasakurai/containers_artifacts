# Success Criteria

- Your team successfully created an RBAC enabled AKS cluster within the address space allocated to you by the network team -- DONE
- Your team successfully redeployed the TripInsights application, now segmented into api and web namespaces, into the cluster -- DONE
- Your team must demonstrate connectivity to and from your cluster by being able to reach the internal-vm (already deployed) -- DONE
- Your team must demonstrate that you are prompted on cluster access to authenticate with AAD -- DONE
- Different members of your team must be able to connect to your cluster using the api-dev and web-dev AAD users and demonstrate appropriate access levels -- DONE

# TODO

- Deploy cluster with nodes in vnet
- redeploy app with new namespaces (API / WEB)
- configure RBAC

# deploy cluster to vnet

https://docs.microsoft.com/en-us/azure-stack/user/kubernetes-aks-engine-custom-vnet?view=azs-2102

## create identiy

az identity create \
 --name aks-identity \
 --resource-group teamResources

{
"clientId": "667a7e29-7ac2-4fed-a790-c6d16ba07743",
"clientSecretUrl": "https://control-northeurope.identity.azure.net/subscriptions/a75d3c2d-d462-47ec-aee4-5efd7f25dd23/resourcegroups/teamResources/providers/Microsoft.ManagedIdentity/userAssignedIdentities/aks-identity/credentials?tid=56a831ed-4f17-4a42-8c08-ad7a78591f7f&oid=7e42ccf3-a4cd-49cb-9fc4-8c0c1c993360&aid=667a7e29-7ac2-4fed-a790-c6d16ba07743",
"id": "/subscriptions/a75d3c2d-d462-47ec-aee4-5efd7f25dd23/resourcegroups/teamResources/providers/Microsoft.ManagedIdentity/userAssignedIdentities/aks-identity",
"location": "northeurope",
"name": "aks-identity",
"principalId": "7e42ccf3-a4cd-49cb-9fc4-8c0c1c993360",
"resourceGroup": "teamResources",
"tags": {},
"tenantId": "56a831ed-4f17-4a42-8c08-ad7a78591f7f",
"type": "Microsoft.ManagedIdentity/userAssignedIdentities"
}

az identity list --query "[].{Name:name, Id:id, Location:location}" -o table

## create cluster

az aks create \
 --resource-group teamResources \
 --name team4vnetcluster \
 --node-count 3 \
 --max-pods 30 \
 --enable-addons monitoring \
 --generate-ssh-keys \
 --network-plugin azure \
 --enable-aad \
 --enable-rbac \
 --attach-acr registryygo2491 \
 --assign-identity /subscriptions/a75d3c2d-d462-47ec-aee4-5efd7f25dd23/resourcegroups/teamResources/providers/Microsoft.ManagedIdentity/userAssignedIdentities/aks-identity \
 --vnet-subnet-id /subscriptions/a75d3c2d-d462-47ec-aee4-5efd7f25dd23/resourceGroups/teamResources/providers/Microsoft.Network/virtualNetworks/vnet/subnets/aks-subnet

az aks get-credentials \
 --resource-group teamResources \
 --name team4vnetcluster \
 --admin

## setup RBAC

### create groups and role assignments in azure

AKS_ID=$(az aks show \
 --resource-group teamResources \
 --name team4vnetcluster \
 --query id -o tsv)

APIDEV_ID=$(az ad group create --display-name api-dev --mail-nickname api-dev --query objectId -o tsv)
az role assignment create \
 --assignee $APIDEV_ID \
 --role "Azure Kubernetes Service Cluster User Role" \
 --scope $AKS_ID

WEBDEV_ID=$(az ad group create --display-name web-dev --mail-nickname web-dev --query objectId -o tsv)
az role assignment create \
 --assignee $WEBDEV_ID \
 --role "Azure Kubernetes Service Cluster User Role" \
 --scope $AKS_ID

https://docs.microsoft.com/en-us/azure/aks/manage-azure-rbac

AKS_ADMIN=$(az ad group create --display-name aks-admin --mail-nickname aks-admin --query objectId -o tsv)

az role assignment create \
 --assignee $AKS_ADMIN \
 --role "Azure Kubernetes Service RBAC Cluster Admin" \
 --scope $AKS_ID

### create role in AKS

az ad group show --group api-dev --query objectId -o tsv
efc9776b-d7d8-47c7-b4ca-c4d7981b625a

az ad group show --group web-dev --query objectId -o tsv
bfc6fb38-d4d1-49f9-8e3f-3af7783e6215
