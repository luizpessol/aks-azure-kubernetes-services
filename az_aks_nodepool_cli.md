# Gerenciamento dos node pools do Cluster (AKS) Azure Kubernetes Services

## Exibir configurações do Node pools

```
az aks nodepool show \
--resource-group rg-aks-cni-cli \
--cluster-name aks-cni-cli-1926 \
--name nodepool1
```

## Adicionar um novo Node pools

```
az aks nodepool add \
    --resource-group rg-aks-cni-cli \
    --cluster-name aks-cni-cli-1926 \
    --name system \
    --node-count 1 \
    --node-vm-size Standard_B2s \
    --node-taints CriticalAddonsOnly=true:NoSchedule \
    --zones 1 2 3 \
    --mode System
```

## Atualizar um Node pools

```
az aks nodepool update \
--resource-group rg-aks-cni-cli \
--cluster-name aks-cni-cli-1926 \
--name user \
--mode System
```

## Deletar um Node pools User ou System

```
az aks nodepool delete \
--resource-group rg-aks-cni-cli \
--cluster-name aks-cni-cli-1926 \
--name user
```

## Adicionar Node pools de VMs Spot

```
az aks nodepool add \
--resource-group rg-aks-cni-cli \
--cluster-name aks-cni-cli-1926 \
--name userpoolspot \
--priority Spot \
--eviction-policy Delete \
--spot-max-price -1 \
--enable-cluster-autoscaler \
--min-count 1 \
--max-count 3 \
--zones 1 2 3 \
--no-wait
```
