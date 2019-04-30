# bamboo-kubernetes-demo

Test Azure Kubernetes Service

## API List

* /api/values
* /api/ip [return thread sleep 5s]

## Docker build image

``` sh
docker build Dockerfile .
```

## AKS Command

### 0, Change the azure region to China

``` sh
az cloud set -n AzureChinaCloud
```

### 1, Push local image to Azure Container Registry (ACR)

#### Create resource group (RG)

``` sh
 az group create -n RG_NAME -l LOCATION
```

#### Create azure container registry

``` sh
az acr create -n ACR-NAME -g RG_NAME -l LOCATION --sku SKU_LEVEL
```

#### List azure container registry

``` sh
az acr list -o table
```

#### Login azure container registry

``` sh
az acr login -n ACR_NAME
```

#### List local docker images

``` sh
docker image list
```

#### Set image tag (format: [ACR_LOGIN_SERVER]/[IMAGE_NAME]:[TAG])

``` sh
docker tag IMAGE_NAME:TAG ACR_SERVER/IMAGE_NAME:TAG
```

#### Push image to ACR

``` sh
docker push ACR_SERVER/IMAGE_NAME:TAG
```

#### List images from ACR

``` sh
az acr repository list -n ACR_NAME
```

### 2, Create Azure Kubernetes Service Cluster (AKS)

#### Create service principle for AKS access ACR (SP)

``` sh
az ad sp create-for-rbac --skip-assignment
```

Result:

``` json
{
  "appId": "SP_APPID",
  "displayName": "SP_DISPLAY_NAME",
  "name": "SP_NAME",
  "password": "SP_PASSWORD",
  "tenant": "SP_TENANT"
}
```

#### Grant AKS access to ACR

``` sh
# Get ACR registry resource id
ACR_ID=$(az acr show --name ACR_NAME --resource-group RG_NAME --query "id" --output tsv)

# Create role assignment [role: acrpull]
az role assignment create --assignee "SP_APPID" --role acrpull --scope $ACR_ID
```

#### Use SP to create AKS cluster (CLS)

``` sh
az aks create --name CLS_NAME --resource-group RG_NAME --node-count 1 --generate-ssh-keys --service-principal "SP_APPID" --client-secret "SP_PASSWORD"
```

#### Connect to AKS cluster

``` sh
az aks get-credentials --name CLS_NAME -g RG_NAME
```

#### Get nodes status

``` sh
kubectl get nodes
```

### 3, Deploy application to Azure Kubernetes Service

#### Get ACR loginServer info

``` sh
az acr list -g RG_NAME
```

#### Deploy application (Please reference .\aks-deployment-template.yml)

``` sh
kubectl apply -f aks-deployment.yml
```

#### Watch service

``` sh
kubectl get service --watch
```