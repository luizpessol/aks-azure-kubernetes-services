# aks-azure-kubernetes-services
## Kubernetes Dashboard

Acesse: https://github.com/kubernetes/dashboard

Obs. Veja a última versão

## Acesso Dashboard 2 opções:

1. Pode usar kubeconfig file;
2. Criar SA token.

## RBAC, copie e cole o comando abaixo em seu terminal:

```
cat > ./dashboard-rbac.yml <<EOF
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dashboard-admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: dashboard-admin-user
  namespace: kubernetes-dashboard

EOF
```

## Apply manifest.

```
kubectl apply -f ./dashboard-rbac.yml
```

## Get token

```
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secrets |grep dashboard-admin-user-token | awk '{print $1}')
```

## Access the dashboard

```
kubectl proxy
```

Access: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

