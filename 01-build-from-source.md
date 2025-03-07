
git clone https://github.com/Azure-Samples/acr-build-helloworld-node
cd acr-build-helloworld-node

ACR_NAME=

RES_GROUP=$ACR_NAME # Resource Group name

az group create --resource-group $RES_GROUP --location westeurope
az acr create --resource-group $RES_GROUP --name $ACR_NAME --sku Standard --location westeurope

az acr build --registry $ACR_NAME --image helloacrtasks:v1 .

AKV_NAME=$ACR_NAME-vault

az keyvault create --resource-group $RES_GROUP --name $AKV_NAME

# Create service principal, store its password in AKV (the registry *password*)
az keyvault secret set \
  --vault-name $AKV_NAME \
  --name $ACR_NAME-pull-pwd \
  --value $(az ad sp create-for-rbac \
                --name $ACR_NAME-pull \
                --scopes $(az acr show --name $ACR_NAME --query id --output tsv) \
                --role acrpull \
                --query password \
                --output tsv)


# Store service principal ID in AKV (the registry *username*)
az keyvault secret set \
    --vault-name $AKV_NAME \
    --name $ACR_NAME-pull-usr \
    --value $(az ad sp list --display-name $ACR_NAME-pull --query [].appId --output tsv)

# Deploy a container

az container create \
    --resource-group $RES_GROUP \
    --name acr-tasks \
    --image $ACR_NAME.azurecr.io/helloacrtasks:v1 \
    --registry-login-server $ACR_NAME.azurecr.io \
    --registry-username $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-usr --query value -o tsv) \
    --registry-password $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-pwd --query value -o tsv) \
    --dns-name-label acr-tasks-$ACR_NAME \
    --query "{FQDN:ipAddress.fqdn}" \
    --output table

# Logs

az container attach --resource-group $RES_GROUP --name acr-tasks

# Cleanup

az group delete --resource-group $RES_GROUP
az ad sp delete --id http://$ACR_NAME-pull