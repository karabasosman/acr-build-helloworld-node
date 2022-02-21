ACR_NAME=    # The name of your Azure container registry
GIT_USER=XXXXX     # Your GitHub user account name
GIT_PAT=XXXXX # The PAT you generated in the previous section

az acr task create \
    --registry $ACR_NAME \
    --name multi-step-task \
    --context https://github.com/$GIT_USER/acr-build-helloworld-node.git#main \
    --file taskmulti.yaml \
    --git-access-token $GIT_PAT