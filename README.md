# hackconf-infra

In this repository you will find all the necessary information about and step by step guide for **Micro service lifecycle - from code to production** workshop

## Prerequisites
This workshop will cover the creation of a micro services application, testing and deployment. For this demonstration we will use free tools and accounts.

##### Accounts
- GitHub - https://github.com/join
- MS Azure - https://azure.microsoft.com/en-us/free/
- Docker Hub - https://hub.docker.com/signup

##### Tools
- kubectl - https://kubernetes.io/docs/tasks/tools/install-kubectl/
- helm -  https://helm.sh/docs/using_helm/#installing-helm
- travis - https://github.com/travis-ci/travis.rb#installation

## Kubernetes

##### Registering AKS provider for the subscription
By default new Azure subscriptions need to have resource provider registered in order to use them.
- Open azure cli
- RUN 
```#!/usr/bin/env bash

SUBSCRIPTIONNAMEORID=YOURSUBSCRIPTIONID
echo $SUBSCRIPTIONNAMEORID

az account set --subscription $SUBSCRIPTIONNAMEORID

NOTREGISTEREDNAMESPACES=$(az provider list --query "[?registrationState=='NotRegistered'].namespace" -o tsv)

for NAMESPACE in $NOTREGISTEREDNAMESPACES
do
echo "Register: $NAMESPACE"
  az provider register --namespace $NAMESPACE
done

echo 'Finished!'
``` 
This script will register all resource provider to your subscription.
<br/> Ref: https://pascalnaber.wordpress.com/2017/05/30/fixing-the-subscription-is-not-registered-to-use-namespace-microsoft-xxx/

##### Creating the cluster
- Open the Azure portal
- Navigate to Kubernetes services
- Click Add
- Select or Create new Resource group
- Choose cluster name and DNS prefix
- Set the Node count to 1
- Click on next
- Make sure Scaling is disabled (We will not use scaling in this demo)
- Click on next
- Select or create new Service Principal
- Click on next
- Activate HTTP application routing
- Click on next 
- Turn off monitoring
- Click on next
- You can tag the cluster if you want (it is not necessary)
- Click on next
- Click on create

##### Retrieving the cluster's kubeconfig
- Open azure cli
- Run `az aks get-credentials --resource-group ${RESOURCE_GROUP_NAME} --name ${KUBERNETES_NAME} --admin` 
  - Print the file returned by the previous cmd `cat /home/user/.kube/config`
  - Copy the file locally
<br/> Ref: https://docs.microsoft.com/en-gb/azure/aks/control-kubeconfig-access

##### Install tiller (helm) on the cluster
- Export the kubeconfig `export KUBECONFIG=${FULL_PATH_TO_KUBECONFIG}`
- Navigate to hackconf-infra/helm
- Run `kubectl apply -f helm-rbac.yaml`
- Run `helm init --history-max 200 --service-account tiller --node-selectors "beta.kubernetes.io/os=linux"`
<br/> Ref: https://docs.microsoft.com/en-gb/azure/aks/kubernetes-helm