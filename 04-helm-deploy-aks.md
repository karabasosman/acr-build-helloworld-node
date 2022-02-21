helm create helloworld   
az aks create -g xx -n xxx --attach-acr xxx

az aks get-credentials -n xx -g xxx
helm install helloworld helloworld


export HELM_EXPERIMENTAL_OCI=1

helm package .

helm registry login $ACR_NAME.azurecr.io \
  --username $ACR_NAME

helm push helloworld-0.1.0.tgz oci://$ACR_NAME.azurecr.io/helm