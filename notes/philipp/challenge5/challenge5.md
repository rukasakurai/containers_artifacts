# success criteria

- Your team secured your Azure SQL Server connection information such that literal values cannot be inappropriately accessed -- WIP
- Your team used an external key vault to store and access secrets inside your cluster, and ensured that access does not require a secret stored in the cluster -- WIP
- Your team ensured that all links on the Trip Viewer site are reachable
  -- user-java is not working, poi seems broken due to key vault work
- Your team ensured the simulator can successfully update the values in the application across all services -- TBD

# set up ingress controller

https://docs.microsoft.com/en-us/azure/application-gateway/tutorial-ingress-controller-add-on-existing

az network public-ip create \
-n aks-ingress-pip \
-g teamResources \
--allocation-method Static \
--sku Standard

## create separate network and gateway ?!?

        az network vnet create \
        -n myVnet \
        -g myResourceGroup \
        --address-prefix 11.0.0.0/8 \
        --subnet-name mySubnet \
        --subnet-prefix 11.1.0.0/16

az network application-gateway create \
-n aks-ingress-gateway \
-l northeurope \
-g teamResources \
--sku Standard_v2 \
--public-ip-address aks-ingress-pip \
--vnet-name vnet \
--subnet ingress-subnet

appgwId=$(az network application-gateway show -n aks-ingress-gateway -g teamResources -o tsv --query "id")

az aks enable-addons \
-n team4vnetcluster \
-g teamResources \
-a ingress-appgw \
--appgw-id $appgwId

## peer the networks ?!?

        nodeResourceGroup=$(az aks show -n myCluster -g myResourceGroup -o tsv --query "nodeResourceGroup")
        aksVnetName=$(az network vnet list -g $nodeResourceGroup -o tsv --query "[0].name")

        aksVnetId=$(az network vnet show -n $aksVnetName -g $nodeResourceGroup -o tsv --query "id")
        az network vnet peering create -n AppGWtoAKSVnetPeering -g myResourceGroup --vnet-name myVnet --remote-vnet $aksVnetId --allow-vnet-access

        appGWVnetId=$(az network vnet show -n myVnet -g myResourceGroup -o tsv --query "id")
        az network vnet peering create -n AKStoAppGWVnetPeering -g $nodeResourceGroup --vnet-name $aksVnetName --remote-vnet $appGWVnetId --allow-vnet-access

## deployment yaml

apiVersion: v1
kind: Pod
metadata:
name: aspnetapp
labels:
app: aspnetapp
spec:
containers:

- image: "mcr.microsoft.com/dotnet/core/samples:aspnetapp"
  name: aspnetapp-image
  ports:
  - containerPort: 80
    protocol: TCP

---

apiVersion: v1
kind: Service
metadata:
name: aspnetapp
spec:
selector:
app: aspnetapp
ports:

- protocol: TCP
  port: 80
  targetPort: 80

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
name: aspnetapp
annotations:
kubernetes.io/ingress.class: azure/application-gateway
spec:
rules:

- http:
  paths:
  - path: /
    backend:
    service:
    name: aspnetapp
    port:
    number: 80
    pathType: Exact
