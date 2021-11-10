# success criteria

- Your team successfully created an AKS cluster in Azure -- DONE
- Your team must demonstrate that at least one pod for each component of the TripInsights application is running -- DONE
- Your team must demonstrate that the components in your cluster can connect to other components or resources: tripviewer is able to access the trips and userprofile services -- DONE
- All APIs are able to access the SQL database provided in your Azure subscription -- DONE
- The POI and User (Java) APIs are reachable from the TripViewer web app top links (but the APIs do not have to be accessible from outside the AKS cluster) -- DONE
- Your team must demonstrate that the components in your cluster are accessing SQL connection information via a Kubernetes Secret -- DONE

# create AKS

az aks create \
 --resource-group teamResources \
 --name team4cluster \
 --node-count 2 \
 --generate-ssh-keys \

az aks get-credentials --resource-group teamResources --name team4-aks

kubectl config set-context team4-aks

kubectl config set-context --current --namespace=sandbox

az aks update -n team4-aks -g teamResources --attach-acr registryygo2491

kubectl exec --stdin --tty <podName> -- /bin/bash

# kubernetes secrets

kubectl create secret generic empty-secret --from-file=.
kubectl get secret empty-secret
kubectl edit secrets empty-secret

# test URLs

http://20.67.174.106:8082/api/poi
http://20.67.201.92:8092/api/trips
http://20.82.174.147:8072/api/user-java/healthcheck
http://20.93.54.66:8062/api/user
http://20.93.54.171:8002/UserProfile
