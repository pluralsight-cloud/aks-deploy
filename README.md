# Deploy Azure Kubernetes Service (AKS)

## Module 1: Deploy and Access an AKS Cluster

### Deploying Dev/Test Clusters

- [What is an Azure landing zone?](https://aka.ms/alz)

### Demo: Deploy a Dev/Test Cluster

- No demo files.

### Deploying Production Clusters

- [AKS landing zone accelerator](https://learn.microsoft.com/azure/cloud-adoption-framework/scenarios/app-platform/aks/landing-zone-accelerator)
- [AKS-Landing-Zone-Accelerator](https://github.com/Azure/AKS-Landing-Zone-Accelerator)
- [Azure Verified Modules (AVM)](https://https://aka.ms/avm)
- [Legacy - CARML Common Azure Resource Modules Library](https://github.com/Azure/ResourceModules)

### Demo: Deploy a Production Cluster

- [AKS Construction Helper](https://github.com/Azure/Aks-Construction)

### Demo: Deploy an AKS Automatic Cluster

- No demo files.

### Demo: Access the AKS Cluster API

- [Demo Files](/ARM%20and%20Bicep%20Templates/) used in the Cloud Playground Sandbox.

- Command to install `kubectl` in the Azure CLI if running locally: `az aks install-cli`

- To make running `kubectl` commands easier, you can set an alias for kubectl using `alias k=kubectl`

- Command used to retrieve credentials for an AKS Cluster:

```bash
az aks get-credentials --resource-group <RG Name> --name <Cluster Name>
```

- Command to get contexts:

```bash
k config get-contexts
```

To switch your current context to a different cluster, you can use 

```bash
k config use-context <context name>
```

## Module 2: Configure AKS Authentication and Authorization

### Demo: Configure AKS Namespaces

- [Demo Files](/ARM%20and%20Bicep%20Templates/) used in the Cloud Playground Sandbox.

### AKS Authentication and Authorization

- No demo files.

### Demo: Configure Microsoft Entra Integration for AKS

- [Demo Files](/ARM%20and%20Bicep%20Templates/) used in the Cloud Playground Sandbox.

- Command used in Cloud Playground to enable both Entra Integration and Azure RBAC for an existing AKS Cluster, while also disabling local accounts:

```bash
az aks update -g <RG Name> -n <TAB Complete Cluster Name> --enable-aad --enable-azure-rbac --disable-local-accounts
```

### Demo: Configure Azure RBAC to Authorize Access to AKS Resources

- No demo files.

### Demo: Configure Azure RBAC for Kubernetes Authorization

- Command used to assign granular permissions at the namespace level using the Azure CLI

```bash
RESOURCE_GROUP=
CLUSTER_NAME=
AKS_ID=$(az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --query id --output tsv)
USER_ID=<Replace with your User ID>
az role assignment create --role "Azure Kubernetes Service RBAC Writer" --assignee $USER_ID  --scope $AKS_ID/namespaces/allowed-namespace
```

- Pod specification used to demonstrate granular permissions at the namespace scope:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: denied-namespace
spec:
  containers:
  - image: busybox
    command:
      - sleep
      - "3600"
    name: busybox
  restartPolicy: Always
```

### Demo: Configure Kubernetes RBAC

- Command used to create a service account

```bash
k create serviceaccount reader-service --namespace kubernetes-rbac
```

- Command used to create a role

```bash
k create role reader-role --verb=get,list,watch --resource=* --namespace kubernetes-rbac
```

- Command used to bind role to service account using a role binding:

```bash
k create rolebinding reader-service-reader-role --role reader-role --serviceaccount kubernetes-rbac:reader-service --namespace kubernetes-rbac
```

- Sample pod specification used in this lesson:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  namespace: kubernetes-rbac
spec:
  serviceAccountName: reader-service
  containers:
    - name: alpinek8s
      image: alpine/k8s:1.28.13
      command:
        - /bin/sh
        - "-c"
        - "sleep infinity"
```

- Command used to apply the pod specification:

```sh
k apply -f pod.yaml
```

- Command used to connect to the pod with an interactive terminal:

```sh
k exec test-pod --namespace kubernetes-rbac -i -t -- sh
```