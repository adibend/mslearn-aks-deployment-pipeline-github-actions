Please deploy AKS service. - done
Have an Azure DevOps \ GitHub pipeline which deploys Ingress Controller, Service and Deployment. - done
Have an auto scaling mechanism in place for increasing Pods up to 10 when reaching a X utilization. - done
Be ready to explain to us the different components, how they work, how we can debug and troubleshoot them
Optional: use Network Policy, HELM

--------------------------------------
--Deploy an aks and a sample application in it
--https://docs.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-portal?tabs=azure-cli
az aks get-credentials --resource-group myResourceGroup --name aks-cls-sandbox-01

kubectl apply -f azure-vote.yaml
kubectl get service azure-vote-front --watch

--Updated the auto-scaler on agentpool to 10  
--https://docs.microsoft.com/en-us/azure/aks/cluster-autoscaler

az aks nodepool update \
  --resource-group myResourceGroup \
  --cluster-name aks-cls-sandbox-01 \
  --name agentpool \
  --update-cluster-autoscaler \
  --min-count 1 \
  --max-count 10

--updated profile scan-interval=30s  
az aks update \
  --resource-group myResourceGroup \
  --name aks-cls-sandbox-01 \
  --cluster-autoscaler-profile scan-interval=30s
  
--horizontal auto-scaler
--https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-scale?tabs=azure-cli#autoscale-pods

az aks show --resource-group myResourceGroup --name aks-cls-sandbox-01 --query kubernetesVersion --output table

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml

--autscale to 10 by cpu-percent of 50%
kubectl autoscale deployment azure-vote-front --cpu-percent=50 --min=3 --max=10
kubectl get hpa


#--------------------------------------
#ngnix ingress example helm deployment as appears in https://www.eksworkshop.com/beginner/060_helm/helm_nginx/
#--------------------------------------

helm repo add stable https://charts.helm.sh/stable
helm repo update
helm search repo nginx
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo bitnami/nginx
helm install mywebserver bitnami/nginx
kubectl get ingress,svc,po,deploy

--Deploying HELM Charts with azure devops pipelines
--https://blog.baeke.info/2021/02/25/deploying-helm-charts-with-azure-devops-pipelines/
--https://www.visualstudiogeeks.com/devops/helm/deploying-helm-chart-with-azdo

ACR

acr name: acrsandboxtest01
az acr repository list --name acrsandboxtest01

az aks list -o table

chartpull
cSz8dMLNB2oIMCwNg0dJW=q5I8Yn1G3k
docker login -u helmdemopull -p cSz8dMLNB2oIMCwNg0dJW=q5I8Yn1G3k acrsandboxtest01.azurecr.io

chartpush
s1DX2nDjo7GL=v2HRkUjQbXY6krxvcsZ
docker login -u helmdemopush -p s1DX2nDjo7GL=v2HRkUjQbXY6krxvcsZ acrsandboxtest01.azurecr.io


--azure pipelines install runner + running sample pipeline
--https://docs.microsoft.com/en-us/azure/devops/pipelines/create-first-pipeline?view=azure-devops&tabs=java%2Ctfs-2018-2%2Cbrowser

adistoken
gm7pw52a2lh66e7gnh7fypswvsbsldacty3sgzsqi6mqlhfjzona

--local agent install and configure
--https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-windows?view=azure-devops#permissions
https://dev.azure.com/adibend69/


az acr create --resource-group myResourceGroup --name acrsandboxtest02 --sku Basic --admin-enabled true
az acr login --name acrsandboxtest02

----------------------
https://www.abhith.net/blog/azure-devops-ci-cd-pipeline-involving-helm-3-acr-aks/
https://cloudblogs.microsoft.com/opensource/2018/11/27/tutorial-azure-devops-setup-cicd-pipeline-kubernetes-docker-helm/
https://docs.microsoft.com/en-us/learn/modules/aks-deployment-pipeline-github-actions/10-exercise-deploy-workflow
----------------------------------
name='aks-cls-sandbox-01'
location='eastus'
rg='myResourceGroup'
acr='acrsandboxtest01'
latestK8sVersion=$(az aks get-versions -l $location --query 'orchestrators[-1].orchestratorVersion' -o tsv)

az aks create -l $location -n $name -g $rg --generate-ssh-keys -k $latestK8sVersion
az aks get-credentials -n $name -g $rg

kubectl create namespace phippyandfriends
kubectl create clusterrolebinding default-view --clusterrole=view --serviceaccount=phippyandfriends:default

ACR_ID=$(az acr show -n $acr -g $rg --query id -o tsv)


MYACR=acrsandboxtest01
az acr create -n $MYACR -g myResourceGroup --sku basic
az aks create -n myAKSCluster -g myResourceGroup --generate-ssh-keys --attach-acr $MYACR

------------------------
https://docs.microsoft.com/en-us/learn/modules/aks-deployment-pipeline-github-actions
https://docs.microsoft.com/en-us/learn/modules/aks-deployment-pipeline-github-actions
---------------------------------

git clone https://github.com/adibend/mslearn-aks-deployment-pipeline-github-actions

--Enviromnet A
Installation concluded, copy these values and store them, you'll use them later in this exercise:
-> Resource Group Name: mslearn-gh-pipelines-7780
-> ACR Name: contosocontainerregistry14973
-> ACR Login Username: contosocontainerregistry14973
-> ACR Password: h77=MmYwV4kcqsU5t9CexQmKYOnTesHE
-> AKS Cluster Name: contosocontainerregistry14973
-> AKS DNS Zone Name: eaea714fc6bc4eb39bcb.eastus.aksapp.io


--Enviromnet B
Installation concluded, copy these values and store them, you'll use them later in this exercise:
-> Resource Group Name: mslearn-gh-pipelines-18560
-> ACR Name: contosocontainerregistry29161
-> ACR Login Username: contosocontainerregistry29161
-> ACR Password: wFn7MuqNtqP4UW2rtTRN5+9lh2q4oYZu
-> AKS Cluster Name: contosocontainerregistry29161
-> AKS DNS Zone Name: e7cea9d0cd954301b921.eastus.aksapp.io


az acr credential show --name contosocontainerregistry29161 --query "username" -o table
az acr credential show --name contosocontainerregistry29161 --query "passwords[0].value" -o table
az acr repository list --name contosocontainerregistry29161 -o table

az acr repository show-tags --repository contoso-website --name contosocontainerregistry29161 -o table

helm install contoso-website ./chart-location --set containerImage="contosocontainerregistry29161/contoso-website"

az aks show -g mslearn-gh-pipelines-7780 -n contosocontainerregistry14973 -o tsv --query addonProfiles.httpApplicationRouting.config.HTTPApplicationRoutingZoneName

----credentials for AWS 
az ad sp create-for-rbac --role Contributor --scopes /subscriptions/86b8be11-5248-4f74-9e30-fbf1217103a5 --sdk-auth

--Enviromnet A
{
  "clientId": "9453739d-e400-49a9-a236-78bbbc5fa6ac",
  "clientSecret": "KVG8Q~CsgqKytITYHckgcSrsXodc4VsT1-66scVB",
  "subscriptionId": "65056ed9-189b-4f50-92ab-10a92e103476",
  "tenantId": "8479da58-18eb-48ff-9c6c-2f92bf4f23f9",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}

--Enviromnet B
{
  "clientId": "f28fd1cb-82cb-4b5b-8f07-d9f53d00a5af",
  "clientSecret": "lei8Q~4waes71TT_iPO~oKXRV3ke2CcoF0VtBbyD",
  "subscriptionId": "86b8be11-5248-4f74-9e30-fbf1217103a5",
  "tenantId": "2434b18a-0608-40cb-b5ad-a99c1b1e35d4",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}

resource-group name: mslearn-gh-pipelines-18560

----ACR_NAME
az acr list --query "[?contains(resourceGroup, 'mslearn-gh-pipelines')].loginServer" -o table
contosocontainerregistry29161.azurecr.io

----ACR_LOGIN
az acr credential show --name contosocontainerregistry29161 --query "username" -o table
contosocontainerregistry29161

----ACR_PASSWORD
az acr credential show --name contosocontainerregistry29161 --query "passwords[0].value" -o table
wFn7MuqNtqP4UW2rtTRN5+9lh2q4oYZu

----DNS
contoso-staging.e7cea9d0cd954301b921.eastus.aksapp.io
contoso-production.e7cea9d0cd954301b921.eastus.aksapp.io


--github working with PAT
PAT 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

https://stackoverflow.com/questions/68775869/message-support-for-password-authentication-was-removed-please-use-a-personal
git config --global user.email "adibend@gmail.com"
git config --global user.name "adibend"

github user for git push:adibend
github apss for git push: is the PAT !!!

git tag -a v2.0.9 -m 'Creating first production deployment 9' && git push --tags

kubectl get po,svc,ingress -n staging
kubectl get po,svc,ingress -n production


--------------------------------------
--Deploy an aks and a sample application in it
--https://docs.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-portal?tabs=azure-cli

az aks get-credentials --resource-group mslearn-gh-pipelines-18560 --name contoso-video

kubectl apply -f azure-vote.yaml
kubectl get service azure-vote-front --watch


--Updated the auto-scaler on agentpool to 10  
--https://docs.microsoft.com/en-us/azure/aks/cluster-autoscaler

az aks nodepool update --enable-cluster-autoscaler --min-count 1 --max-count 5 --resource-group mslearn-gh-pipelines-18560 --name nodepool1 --cluster-name contoso-video
az aks nodepool update --resource-group mslearn-gh-pipelines-18560 --cluster-name contoso-video --name nodepool1 --update-cluster-autoscaler --min-count 1 --max-count 10

--updated profile scan-interval=30s  
az aks update --resource-group mslearn-gh-pipelines-18560 --name contoso-video --cluster-autoscaler-profile scan-interval=30s
  
--horizontal auto-scaler
--https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-scale?tabs=azure-cli#autoscale-pods

az aks show --resource-group mslearn-gh-pipelines-18560 --name contoso-video --query kubernetesVersion --output table

git clone https://github.com/Azure-Samples/azure-voting-app-redis.git
kubectl apply -f azure-vote-all-in-one-redis.yaml
kubectl get service azure-vote-front --watch

--ACR
az acr list -o table
az acr repository list --name contosocontainerregistry29161 --resource-group mslearn-gh-pipelines-18560 -o table

--AKS cluster commands:
az aks list --output table
az aks nodepool list --resource-group mslearn-gh-pipelines-18560 --cluster-name contoso-video --output table
az aks pod-identity list --resource-group mslearn-gh-pipelines-18560 --cluster-name contoso-video --output table


--get kubernetesVersion
az aks show --resource-group mslearn-gh-pipelines-18560 --name contoso-video --query kubernetesVersion --output table

--autscale to 10 by cpu-percent of 50%
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml
kubectl autoscale deployment azure-vote-front --cpu-percent=50 --min=3 --max=10
kubectl get hpa


