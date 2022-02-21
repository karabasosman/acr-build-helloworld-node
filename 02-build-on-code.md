ACR_NAME=    # The name of your Azure container registry
GIT_USER=     # Your GitHub user account name
GIT_PAT= # The PAT you generated in the previous section

az acr task create \
    --registry $ACR_NAME \
    --name taskhelloworld \
    --image helloworld:{{.Run.ID}} \
    --context https://github.com/$GIT_USER/acr-build-helloworld-node.git#main \
    --file Dockerfile \
    --git-access-token $GIT_PAT

az acr task run --registry $ACR_NAME --name taskhelloworld