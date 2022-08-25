# Azure-RBAC-with-AD

## Create Resource Group

```bash
az group create -l northeurope -n azdevops
```
## Create an AKS-managed Azure AD cluster

```bash
az aks create -g azdevops -n aksdemo --node-count 1 --enable-aad --enable-azure-rbac
```

## Get your AKS Resource ID

```bash
AKS_ID=$(az aks show -g azdevops -n aksdemo --query id -o tsv)
```

## Create Group Name

```bash
APPDEV_ID=$(az ad group create --display-name appdev --mail-nickname appdev --query objectId -o tsv)
```

>:note: Apparently it's an issue with running commands in git bash on windows, and the solution seems to be:
export MSYS_NO_PATHCONV=1

## Copy group-name from Azure AD and replace in below command

```bash
az role assignment create --assignee <group-object-name> --role "Azure Kubernetes Service Cluster User Role" --scope $AKS_ID
```

## Create user with below command replace values of principle id and password

```bash
AKSDEV_ID=$(az ad user create \
  --display-name "AKS Dev" \
  --user-principal-name naresh240@principle-id \
  --password Naresh#123 \
  --query objectId -o tsv)
```

## Get user object ID and replace in below comamnd

```bash  
az ad group member add --group appdev --member-id <user-object-id>
```

## Get credentials of aks admin with below command

```bash
az aks get-credentials --resource-group azdevops --name aksdemo --admin
```

## Create namespace under cluster

```bash
kubectl create namespace dev
```

## Create role and role binding for read access

```bash
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
  
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: pod-reader
  namespace: dev
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
subjects:
- kind: Group
  namespace: dev
  name: <group-object-Id>
```  

## Login with user and run command

```bash
az login
```

## Get user config credetials for AKS cluster
```bash
az aks get-credentials --resource-group azdevops --name aksdemo --overwrite-existing
```
