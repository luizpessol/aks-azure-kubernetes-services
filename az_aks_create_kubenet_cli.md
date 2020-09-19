# aks-azure-kubernetes-services

## Criar variaveis de Ambiente, Região, Resource Group, subnet e vnet onde o Cluster será provisionado.

```
SUBSCRIPTION=MSDN
REGION_NAME=eastus
RESOURCE_GROUP=rg-aks-kubenet-cli
SUBNET_NAME=subnet-aks-kubenet-cli
VNET_NAME=vnet-aks
RG_VNET=rg-vnet-aks
```

## Armazenar em uma variável ID da Subnet:

```
SUBNET_ID=$(az network vnet subnet show \
    --resource-group $RG_VNET \
    --vnet-name $VNET_NAME \
    --name $SUBNET_NAME \
    --query id -o tsv)
```

## Armazenar em uma variável a última versão do AKS para criação do Cluster

```
VERSION=$(az aks get-versions \
    --location $REGION_NAME \
    --query 'orchestrators[?!isPreview] | [-1].orchestratorVersion' \
    --output tsv)
```

## Variável para nome do Cluster AKS.
```
AKS_CLUSTER_NAME=aks-kubenet-cli-$RANDOM
```

## Exibir o nome guardado nessa variável que contem o nome do Cluster que deve ser único 
```
echo $AKS_CLUSTER_NAME
```

## Criação do Cluster AKS com o comando az aks create, rede Kubenet (Basic).
```
  az aks create \
  --name $AKS_CLUSTER_NAME \
  --resource-group $RESOURCE_GROUP \
  --subscription $SUBSCRIPTION \
  --dns-name-prefix aks-kubenet \
  --dns-service-ip 10.0.0.10 \
  --docker-bridge-address 172.17.0.1/16 \
  --kubernetes-version $VERSION \
  --location $REGION_NAME \
  --network-plugin kubenet \
  --node-osdisk-size 500 \
  --node-vm-size Standard_F8s \
  --pod-cidr 10.244.0.0/16 \
  --service-cidr 10.0.0.0/16 \
  --load-balancer-sku basic \
  --enable-cluster-autoscaler \
  --node-count 2 \
  --min-count 2 \
  --max-count 3 \
  --vnet-subnet-id $SUBNET_ID \
  --ssh-key-value /home/luiz/.ssh/id_rsa.pub \
  --tags 'Ambiente=DEV'
```

## Para validar acesso ao Cluster após criação, gerar kubeconfig.
```
FILE="./$AKS_CLUSTER_NAME.kubeconfig"
```
```
az aks get-credentials \
  -n $AKS_CLUSTER_NAME \
  -g $RESOURCE_GROUP \
  --subscription $SUBSCRIPTION \
  --admin \
  --file $FILE
```
```
export KUBECONFIG=$FILE
```

## Exibir os nodes do Cluster
```
kubectl get nodes
```
