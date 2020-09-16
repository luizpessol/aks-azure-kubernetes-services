# aks-azure-kubernetes-services

# Criar variaveis de Ambiente, Região, Resource Group, subnet e vnet onde o Cluster será provisionado.

SUBSCRIPTION=MSDN
REGION_NAME=eastus
RESOURCE_GROUP=rg-aks-cni-cli
SUBNET_NAME=subnet-aks-cni-cli
VNET_NAME=vnet-aks
RG_VNET=rg-vnet-aks


# Armazenar em uma variável ID da Subnet:

SUBNET_ID=$(az network vnet subnet show \
    --resource-group $RG_VNET \
    --vnet-name $VNET_NAME \
    --name $SUBNET_NAME \
    --query id -o tsv)

# Armazenar em uma variável a ultima versão do AKS para criação do Cluster

VERSION=$(az aks get-versions \
    --location $REGION_NAME \
    --query 'orchestrators[?!isPreview] | [-1].orchestratorVersion' \
    --output tsv)

# Variável para nome do Cluster AKS

AKS_CLUSTER_NAME=aks-cni-cli-$RANDOM

# Exibir o nome guardado nessa variável que contem o nome do Cluster que deve ser único 

echo $AKS_CLUSTER_NAME

# Vamos criar uma service principal com nome customizado:

az ad sp create-for-rbac \
  --skip-assignment \
  -n "sp-aks-dev"

Salvar as informações para utilização:

{
  
}

# Adicionar permissão no RG Vnet para service principal do AKS:

az role assignment create \
  --role "Owner" \
  --assignee "id_service_principal" \
  --resource-group $RG_VNET

# Criação do Cluster AKS com o comando az aks create, rede Azure CNI

az aks create \
  --name $AKS_CLUSTER_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $REGION_NAME \
  --kubernetes-version $VERSION \
  --subscription $SUBSCRIPTION \
  --ssh-key-value /home/lpessol/.ssh/id_rsa.pub \
  --service-principal "service_principal_id" \
  --client-secret "service_principal_password" \
  --network-plugin azure \
  --load-balancer-sku standard \
  --outbound-type loadBalancer \
  --vnet-subnet-id $SUBNET_ID \
  --service-cidr 10.0.0.0/16 \
  --dns-service-ip 10.0.0.10 \
  --docker-bridge-address 172.17.0.1/16 \
  --node-vm-size Standard_B2s \
  --enable-cluster-autoscaler \
  --max-count 5 \
  --min-count 2 \
  --node-count 2 \
  --tags 'Ambiente=DEV'

# Para validar acesso ao Cluster após criação, gerar kubeconfig

FILE="./$AKS_CLUSTER_NAME.kubeconfig"

az aks get-credentials \
  -n $AKS_CLUSTER_NAME \
  -g $RESOURCE_GROUP \
  --subscription $SUBSCRIPTION \
  --admin \
  --file $FILE
  
export KUBECONFIG=$FILE

# Exibir os nodes do Cluster

kubectl get nodes
