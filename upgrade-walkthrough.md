# Upgrade Walkthrough

In this walkthrough we'll show the process for upgrading an AKS Engine cluster on Azure Stack Hub, starting with an AKS Engine upgrade.

Our starting cluster will be the following:
* AKS Engine: v0.55.4
* Kubernetes: v1.15.12
* OS: Ubuntu 16.04.7 LTS   4.15.0-1095-azure

## Deploy Sample Cluster

First lets create our sample cluster.

```bash
# Install the proper version of AKS Engine
curl -o get-akse.sh https://raw.githubusercontent.com/Azure/aks-engine/master/scripts/get-akse.sh
chmod 700 get-akse.sh
./get-akse.sh --version v0.55.4

# Check your version
aks-engine version

Version: v0.55.4
GitCommit: 0b2df8a7a
GitTreeState: clean
```

Next we pull down a matching sample api model file.

```bash
# Download the file locally
wget https://raw.githubusercontent.com/Azure/aks-engine/v0.55.0/examples/azure-stack/kubernetes-azurestack.json

# Edit to update with your location, portal URL, cluster dns name, ssh key and client credentials
nano kubernetes-azurestack.json
```

Deploy the cluster.

```bash
aks-engine deploy \
--azure-env AzureStackCloud \
--location <Insert Location ID> \
--resource-group <Insert Target Resource Group> \
--api-model ./kubernetes-azurestack.json \
--output-directory <Output Directory Name> \
--client-id "<Insert Client ID>" \
--client-secret "<Insert Client Secret>" \
--subscription-id "<Insert Target Subscription ID>"

# Once deployed, check the version
kubectl get nodes -o wide

NAME                       STATUS   ROLES    AGE   VERSION    INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
k8s-linuxpool-42499568-0   Ready    agent    35m   v1.15.12   10.240.0.4     <none>        Ubuntu 16.04.7 LTS   4.15.0-1095-azure   docker://19.3.12
k8s-linuxpool-42499568-1   Ready    agent    35m   v1.15.12   10.240.0.6     <none>        Ubuntu 16.04.7 LTS   4.15.0-1095-azure   docker://19.3.12
k8s-linuxpool-42499568-2   Ready    agent    35m   v1.15.12   10.240.0.5     <none>        Ubuntu 16.04.7 LTS   4.15.0-1095-azure   docker://19.3.12
k8s-master-42499568-0      Ready    master   36m   v1.15.12   10.240.255.5   <none>        Ubuntu 16.04.7 LTS   4.15.0-1095-azure   docker://19.3.12
k8s-master-42499568-1      Ready    master   35m   v1.15.12   10.240.255.6   <none>        Ubuntu 16.04.7 LTS   4.15.0-1095-azure   docker://19.3.12
k8s-master-42499568-2      Ready    master   35m   v1.15.12   10.240.255.7   <none>        Ubuntu 16.04.7 LTS   4.15.0-1095-azure   docker://19.3.12
```

Let's also deploy a sample app we can watch during the upgrade process:

```bash
# Clone this repository
git clone https://github.com/swgriffith/aks-on-stack.git
cd aks-on-stack

# Deploy the namespace and application components
kubectl apply -f sample-app/namespace.yaml
kubectl apply -f sample-app/mongodb.yaml
kubectl apply -f sample-app/data-api.yaml
kubectl apply -f sample-app/flights-api.yaml
kubectl apply -f sample-app/quakes-api.yaml
kubectl apply -f sample-app/weather-api.yaml
kubectl apply -f sample-app/service-tracker-ui.yaml

# Watch the services and pods come online
watch kubectl get svc,pods -o wide -n service-tracker

# Once the 'EXTERNAL-IP' field is populated for the service-tracker-ui
# copy the IP and open your browser to http://<EXTERNAL-IP>:8080
# Click the 'Refresh Data' links to load the dashboard
```

---

## Review our options

We want to upgrade in a way that has minimal impact to our application and runtime, so before we do anything we need to understand the upgrade options we have from the following:

* AKS Engine: v0.55.4
* Kubernetes: v1.15.12
* OS: Ubuntu 16.04.7 LTS   4.15.0-1095-azure

Checking the AKS Engine on Azure Stack Hub [release notes](https://docs.microsoft.com/en-us/azure-stack/user/kubernetes-aks-engine-release-notes?view=azs-2102#aks-engine-and-corresponding-image-mapping) we can see that the next supported version of AKS Engine is v0.60.1....but that version does not support Kubernetes 1.15. So, before we can upgrade AKS Engine versions to take adavantage of new OS images and Azure features, we need to get up to a supported Kubernetes version.

Let's see what options we have:

```bash
# Use the aks-engine get-versions command to see what we can upgrade to
aks-engine get-versions --version 1.15.12
Version Upgrades
1.15.12 1.16.13, 1.16.14
```

Great, we can go from our current version up to 1.16.14, which will get us into the Kubernetes minor release of 1.16, and that version is supported by AKS Engine v0.60.1.

## Initial Kubernetes Upgrade

We'll use the aks-engine upgrade command to bump our Kubernetes version from 1.15.12 to 1.16.14. While we're at it, we can watch the application we deployed to see how it behaves. 

Open up two terminal windows.

Terminal 1
```bash
# Watch your application pods
# Note that we have two pods for each service, except the mongo DB
watch kubectl get nodes,pods -n service-tracker -o wide

NAME                            STATUS   ROLES    AGE   VERSION    INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
node/k8s-linuxpool-42499568-0   Ready    agent    77m   v1.15.12   10.240.0.4     <none>        Ubuntu 16.04.7 LTS   4.15.0-1095-azure   docker://19.3.12
node/k8s-linuxpool-42499568-1   Ready    agent    77m   v1.15.12   10.240.0.6     <none>        Ubuntu 16.04.7 LTS   4.15.0-1095-azure   docker://19.3.12
node/k8s-linuxpool-42499568-2   Ready    agent    77m   v1.15.12   10.240.0.5     <none>        Ubuntu 16.04.7 LTS   4.15.0-1095-azure   docker://19.3.12
node/k8s-master-42499568-0      Ready    master   77m   v1.15.12   10.240.255.5   <none>        Ubuntu 16.04.7 LTS   4.15.0-1095-azure   docker://19.3.12
node/k8s-master-42499568-1      Ready    master   77m   v1.15.12   10.240.255.6   <none>        Ubuntu 16.04.7 LTS   4.15.0-1095-azure   docker://19.3.12
node/k8s-master-42499568-2      Ready    master   77m   v1.15.12   10.240.255.7   <none>        Ubuntu 16.04.7 LTS   4.15.0-1095-azure   docker://19.3.12

NAME                                      READY   STATUS    RESTARTS   AGE   IP           NODE                       NOMINATED NODE   READINESS GATES
pod/data-api-d545fc46b-4jdd5              1/1     Running   1          57m   10.244.4.3   k8s-linuxpool-42499568-1   <none>           <none>
pod/data-api-d545fc46b-wr8ff              1/1     Running   0          51m   10.244.5.5   k8s-linuxpool-42499568-2   <none>           <none>
pod/flights-api-6d669fcbd4-6brqz          1/1     Running   0          57m   10.244.3.4   k8s-linuxpool-42499568-0   <none>           <none>
pod/flights-api-6d669fcbd4-bvzl9          1/1     Running   0          51m   10.244.5.6   k8s-linuxpool-42499568-2   <none>           <none>
pod/flights-api-6d669fcbd4-f44hn          1/1     Running   0          51m   10.244.4.4   k8s-linuxpool-42499568-1   <none>           <none>
pod/mongodb-768cbb678d-hfmnz              1/1     Running   0          57m   10.244.5.4   k8s-linuxpool-42499568-2   <none>           <none>
pod/quakes-api-5f9f7f4875-lxnw6           1/1     Running   0          57m   10.244.3.3   k8s-linuxpool-42499568-0   <none>           <none>
pod/quakes-api-5f9f7f4875-xd2zv           1/1     Running   0          51m   10.244.5.7   k8s-linuxpool-42499568-2   <none>           <none>
pod/service-tracker-ui-7977498d47-d72ms   1/1     Running   0          51m   10.244.4.5   k8s-linuxpool-42499568-1   <none>           <none>
pod/service-tracker-ui-7977498d47-n9kkt   1/1     Running   0          57m   10.244.5.3   k8s-linuxpool-42499568-2   <none>           <none>
pod/weather-api-6697b8c94-6fqg8           1/1     Running   0          51m   10.244.3.5   k8s-linuxpool-42499568-0   <none>           <none>
pod/weather-api-6697b8c94-vgxvc           1/1     Running   0          57m   10.244.4.2   k8s-linuxpool-42499568-1   <none>           <none>
```

Terminal 2
```bash
# Upgrade your Kubernetes Version
aks-engine upgrade \
--azure-env AzureStackCloud \
--location <Insert Location ID> \
--resource-group <Insert Target Resource Group> \
--api-model <Insert Path to generated apimodel.json file> \
--client-id "<Insert Client ID>" \
--client-secret "<Insert Client Secret>" \
--subscription-id "<Insert Target Subscription ID>" \
--upgrade-version 1.16.14
```

The upgrade will take a while, as each node is individually upgraded. Keep an eye on the 'watch' running in terminal 1, and you'll start to see nodes moved to 'Not Ready' state and then the node Kubernetes version will switch over to 1.16.14 and the node will be in 'Ready' state again. If you watch the output of aks-engine, you'll see your node count increased by 1 each time a node is upgraded. AKS Engine will add a node to the cluster, get it online, and then remove the old node.

You should also see your application pods coming off and online, but since our deployment as two replicas of each application component, with the exception of MongoDB, there shouldn't be any service interuption until the stateful workloads transition. This is why it's so important to consider the persistence layer of your application, and either move out of cluster, or to a solution in cluster that will manage your trasnsition during upgrades (ex. The [MongoDB Operator](https://docs.mongodb.com/kubernetes-operator/master/)

--- 

## Upgrade AKS Engine

First let's upgrade the AKS Engine version. After checking the docs, we see that the most recent supported version for Azure Stack Hub [documented here](https://docs.microsoft.com/en-us/azure-stack/user/kubernetes-aks-engine-release-notes?view=azs-2102#aks-engine-and-corresponding-image-mapping) is **v0.60.1**, so let's pull that version down.

```bash
# Upgrade AKS Engine versions
curl -o get-akse.sh https://raw.githubusercontent.com/Azure/aks-engine/master/scripts/get-akse.sh
chmod 700 get-akse.sh
./get-akse.sh --version v0.60.1

# Check your version
aks-engine version

Version: v0.60.1
GitCommit: 91d42f2ba
GitTreeState: clean
```

---
## Upgrade Node OS

Looking at the [release motes]() for AKS Engine v0.60.1, we can see that the has been an OS Image update. Let's upgrade our cluster to that updated image.

```bash

```



Next we need to have a look at our api model to see if there were any breaking changes. We could dig through the release notes for this, but the Azure Stack Hub team creates example apimodel files for all of their supported releases, so we can just look [here](https://github.com/Azure/aks-engine/tree/patch-release-v0.60.1/examples/azure-stack) to see any changes in v0.60.1, and compare against our existing file. After comparing, we can see that there was a minor change associated to issue [#4038](https://github.com/Azure/aks-engine/pull/4038), so let's pull the latest and update with our values.

```bash
# Download the file locally
wget https://raw.githubusercontent.com/Azure/aks-engine/patch-release-v0.60.1/examples/azure-stack/kubernetes-azurestack.json

# Edit to update with your location, portal URL, cluster dns name, ssh key and client credentials
nano kubernetes-azurestack.json
```