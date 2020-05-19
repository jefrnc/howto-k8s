
Verifico la conexion con AKS de los Clustes que levante

az aks install-cli

az aks get-credentials --resource-group DevOpsLab-Test-RG --name aks-test

kubectl get nodes
NAME                                STATUS   ROLES   AGE   VERSION
aks-agentpool-30099824-vmss000000   Ready    agent   37h   v1.15.10
aks-agentpool-30099824-vmss000001   Ready    agent   37h   v1.15.10
aks-agentpool-30099824-vmss000002   Ready    agent   37h   v1.15.10



//https://azure.microsoft.com/es-es/resources/templates/docker-rancher/
Deployamos una vm de Ubuntu con Docker Engine y Rancher
  

Línea de comandos
az group create --name <resource-group-name> --location <resource-group-location> #use this command when you need to create a new resource group for your deployment
az group deployment create --resource-group <my-resource-group> --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/docker-rancher/azuredeploy.json



 


Server : https://kvaes.wordpress.com/2016/01/22/deploying-rancher-server-via-an-azure-resource-manager-template/

Nodes : https://kvaes.wordpress.com/2016/01/18/deploying-rancher-hosts-via-an-azure-resource-manager-template/


https://bitbucket.org/kvaes/azure-rancher/src/master/

//primero el nodo y despues el host

En la primer prueba levante erroneamente Rancher quedo de la siguiente manera:


https://40.76.18.33:8443/g/clusters
user: semperti
pwd vm: {7m7T}Ss,EK`DPhS

user: admin
pwd: sQj*Hf3s6Lw\*,$`
https://rancher-mgmt.eastus.cloudapp.azure.com:8443



https://ranchrealnode.eastus2.cloudapp.azure.com/g/clusters

Usuario: admin
Contraseña: *jose271


Creo para Testing

//https://rancher.com/docs/rancher/v2.x/en/cluster-provisioning/hosted-kubernetes-clusters/aks/

az ad sp create-for-rbac --skip-assignment                                                      {
  "appId": "692706eb-ed70-423e-a95e-926d6afdd49e",
  "displayName": "azure-cli-2020-04-24-03-19-28",
  "name": "http://azure-cli-2020-04-24-03-19-28",
  "password": "38c5a202-2fad-4878-ae9f-46229d47efdf",
  "tenant": "69510107-775d-4374-a979-c08f0dc657c5"
}



PS C:\Windows\system32> az account list --refresh --output table                                                        Name                       CloudName    SubscriptionId                        State    IsDefault
-------------------------  -----------  ------------------------------------  -------  -----------
Microsoft Partner Network  AzureCloud   a2a1f0bd-856b-47ae-9a18-9ccdbc674d9a  Enabled  True
PS C:\Windows\system32>         


az role assignment create \
  --assignee $appId \
  --scope /subscriptions/$<SUBSCRIPTION-ID>/resourceGroups/$<GROUP> \
  --role Contributor
 

 Below is an example command for assigning the Contributor role to a service principal. Contributors can manage anything on AKS but cannot give access to others:



az role assignment create  \
  --assignee 692706eb-ed70-423e-a95e-926d6afdd49e  \
  --scope /subscriptions/a2a1f0bd-856b-47ae-9a18-9ccdbc674d9a/resourceGroups/DevOpsLabs-AKS-Test-RG \
  --role Contributor


  az role assignment create  --assignee 692706eb-ed70-423e-a95e-926d6afdd49e --scope /subscriptions/a2a1f0bd-856b-47ae-9a18-9ccdbc674d9a/resourceGroups/DevOpsLabs-AKS-Test --role Contributor


 
{
  "canDelegate": null,
  "id": "/subscriptions/a2a1f0bd-856b-47ae-9a18-9ccdbc674d9a/resourceGroups/DevOpsLabs-AKS-Test-RG/providers/Microsoft.Authorization/roleAssignments/4ee97921-634f-4eac-9d73-acadcced2c4d",
  "name": "4ee97921-634f-4eac-9d73-acadcced2c4d",
  "principalId": "95c7eb07-2f1e-402d-9503-3f08dfd1be56",
  "principalType": "ServicePrincipal",
  "resourceGroup": "DevOpsLabs-AKS-Test-RG",
  "roleDefinitionId": "/subscriptions/a2a1f0bd-856b-47ae-9a18-9ccdbc674d9a/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
  "scope": "/subscriptions/a2a1f0bd-856b-47ae-9a18-9ccdbc674d9a/resourceGroups/DevOpsLabs-AKS-Test-RG",
  "type": "Microsoft.Authorization/roleAssignments"
}


You can also create the service principal and give it Contributor privileges by combining the two commands into one. In this command, the scope needs to provide a full path to an Azure resource:


az ad sp create-for-rbac \
  --scope /subscriptions/$<SUBSCRIPTION-ID>/resourceGroups/$<GROUP> \
  --role Contributor

az ad sp create-for-rbac \
  --scope /subscriptions/a2a1f0bd-856b-47ae-9a18-9ccdbc674d9a/resourceGroups/DevOpsLab-Test-RG \
  --role Contributor

Creating a role assignment under the scope of "/subscriptions/a2a1f0bd-856b-47ae-9a18-9ccdbc674d9a/resourceGroups/DevOpsLab-Test-RG"
  Retrying role assignment creation: 1/36
{
  "appId": "93060046-c48a-487b-aaaa-5574acfdb946",
  "displayName": "azure-cli-2020-04-24-03-26-39",
  "name": "http://azure-cli-2020-04-24-03-26-39",
  "password": "624c1bf0-f811-4a64-8b96-48791d789c9a",
  "tenant": "69510107-775d-4374-a979-c08f0dc657c5"
}


Creo para Produccion
az aks get-credentials --resource-group DevOpsLab-Prod-RG --name aks


PS C:\Windows\system32> az account list --refresh --output table                                                        Name                       CloudName    SubscriptionId                        State    IsDefault
-------------------------  -----------  ------------------------------------  -------  -----------
Microsoft Partner Network  AzureCloud   a2a1f0bd-856b-47ae-9a18-9ccdbc674d9a  Enabled  True
PS C:\Windows\system32>         
 

az role assignment create  \
  --assignee 692706eb-ed70-423e-a95e-926d6afdd49e  \
  --scope /subscriptions/a2a1f0bd-856b-47ae-9a18-9ccdbc674d9a/resourceGroups/DevOpsLab-Prod-RG \
  --role Contributor
{
  "canDelegate": null,
  "id": "/subscriptions/a2a1f0bd-856b-47ae-9a18-9ccdbc674d9a/resourceGroups/DevOpsLab-Prod-RG/providers/Microsoft.Authorization/roleAssignments/8909ef8e-ac61-46bf-bcf3-16d4d3a12220",
  "name": "8909ef8e-ac61-46bf-bcf3-16d4d3a12220",
  "principalId": "95c7eb07-2f1e-402d-9503-3f08dfd1be56",
  "principalType": "ServicePrincipal",
  "resourceGroup": "DevOpsLab-Prod-RG",
  "roleDefinitionId": "/subscriptions/a2a1f0bd-856b-47ae-9a18-9ccdbc674d9a/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
  "scope": "/subscriptions/a2a1f0bd-856b-47ae-9a18-9ccdbc674d9a/resourceGroups/DevOpsLab-Prod-RG",
  "type": "Microsoft.Authorization/roleAssignments"
}



//debemos ir a rancher, importr un nuevo cluster y nos va a dar un yamel bajar el mismo e importarlo en la instancia de k8s

PS C:\Repos\k8s-bootcamp\rancher>  kubectl apply -f rancher.yaml
clusterrole.rbac.authorization.k8s.io/proxy-clusterrole-kubeapiserver created
clusterrolebinding.rbac.authorization.k8s.io/proxy-role-binding-kubernetes-master created
namespace/cattle-system created
serviceaccount/cattle created
clusterrolebinding.rbac.authorization.k8s.io/cattle-admin-binding created
secret/cattle-credentials-d64021b created
clusterrole.rbac.authorization.k8s.io/cattle-admin created
deployment.apps/cattle-cluster-agent created
daemonset.apps/cattle-node-agent created
PS C:\Repos\k8s-bootcamp\rancher>


 
Para importar el AKS de test me baje el yaml ya que me tiraba error de ssl
https://ranchrealnode.eastus2.cloudapp.azure.com/v3/import/p592xbhlhsmzt5sm92hkqj6zxg7mq9x94g5h9wwwd45b54j2bd6dvm.yaml


PS C:\Repos\k8s-bootcamp\rancher>  kubectl apply -f rancher.yaml
clusterrole.rbac.authorization.k8s.io/proxy-clusterrole-kubeapiserver created
clusterrolebinding.rbac.authorization.k8s.io/proxy-role-binding-kubernetes-master created
namespace/cattle-system created
serviceaccount/cattle created
clusterrolebinding.rbac.authorization.k8s.io/cattle-admin-binding created
secret/cattle-credentials-649d7dd created
clusterrole.rbac.authorization.k8s.io/cattle-admin created
deployment.apps/cattle-cluster-agent created
daemonset.apps/cattle-node-agent created