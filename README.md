# AKS Engine on Azure Stack Hub - Best Practices

> **_NOTE:_** The purpose of this document is to provide a light weight guide to AKS on Azure Stack Hub deployment. The best, in depth, source for guidance is the [Azure Documentation](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-2008) which I used to create this summary content. Think of this content as the CliffsNotes™ version.

## Getting started

The first decision to make when looking to run Kubernetes on Azure Stack Hub (henceforth ASH) is to determine which approach you want to take for cluster lifecycle management (i.e. Cluster Create, Update, Scale, Delete). You have various options, including tools that target bare metal and virtual machine based deployments, like [k3s](https://k3s.io/). In this guide our focus will be on the Azure Native options:

* Azure Kubernetes Service Engine
* AKS Resource Provider for Azure Stack Hub 

As the AKS Resource provider is still in preview, we'll be focusing on the AKS Engine option for cluster lifecycle management.

---

## AKS Engine on ASH Support

AKS Engine is an open source project run by Microsoft, so it typically would not have standard support beyond the open source issue process. However, as a mechanism for running AKS on ASH there is support available. For the full details see the [support policy document](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-support?view=azs-2008) provided by the Azure Stack Hub team. 

---

## Pre-reqs: Getting ready to install AKS Engine on Azure Stack Hub

The Azure documentation is very thorough in providing the steps needed to install AKS on Azure Stack Hub, but lets highlight a few key points.

1. Review the prerequisites documented [here](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-set-up?view=azs-2008), which in summary tells you to confirm the following:

    * ASH is on version 1910 or later (See the AKS Engine to ASH version mapping [here](https://docs.microsoft.com/en-us/azure-stack/user/kubernetes-aks-engine-release-notes?view=azs-2008#aks-engine-and-azure-stack-version-mapping))
    * ASH has the Linux Custom Script Extension v2 installed from the Marketplace
    * Service Principal has been created with Contributor Rights on your target ASH Resource Group
    * An SSH private/public key pair is available, which you will pass in the public key on cluster creation to be used at the cluster access token

    | Azure Stack Hub version                    | AKS engine version         |
    |------------------------------------------------|--------------------------------|
    | 1910                                           | 0.43.0, 0.43.1                 |
    | 2002                                           | 0.48.0, 0.51.0                 |
    | 2005                                           | 0.48.0, 0.51.0, 0.55.0, 0.55.4 |
    | 2008                                           | 0.55.4, 0.60.1                 |
<br></br>
2. Choose a version of Kubernetes you'd like to install. If this is your first cluster, you should align with the latest supported version, however you may be limited based on the base OS images available in your ASH Marketplace. Review the following two lists to help you choose.

   * You can see the full list of AKS Engine to base OS image mappings [here](https://docs.microsoft.com/en-us/azure-stack/user/kubernetes-aks-engine-release-notes?view=azs-2008#aks-engine-and-corresponding-image-mapping)
   * You can see details on the AKS Engine and aligned Kubernetes supported versions for Azure Stack Hub [here](https://docs.microsoft.com/en-us/azure-stack/user/kubernetes-aks-engine-release-notes?view=azs-2008#aks-engine-and-azure-stack-version-mapping)


|      AKS engine     |      AKS base image     |      Kubernetes versions     |      API model samples     |
|-|-|-|-|
|     v0.43.1    |     AKS Base Ubuntu 16.04-LTS Image Distro, October 2019   (2019.10.24)    |     1.15.5, 1.15.4, 1.14.8, 1.14.7    |  |
|     v0.48.0    |     AKS Base Ubuntu 16.04-LTS Image Distro, March 2020   (2020.03.19)    |     1.15.10, 1.14.7    |  |
|     v0.51.0    |     AKS Base Ubuntu 16.04-LTS Image Distro, May 2020 (2020.05.13),   AKS Base Windows Image (17763.1217.200513)    |     1.15.12, 1.16.8, 1.16.9    |     [Linux](https://github.com/Azure/aks-engine/blob/v0.51.0/examples/azure-stack/kubernetes-azurestack.json), [Windows](https://github.com/Azure/aks-engine/blob/v0.51.0/examples/azure-stack/kubernetes-windows.json)    |
|     v0.55.0    |     AKS Base Ubuntu 16.04-LTS Image Distro, August 2020   (2020.08.24), AKS Base Windows Image (17763.1397.200820)    |     1.15.12, 1.16.14, 1.17.11    |     [Linux](https://github.com/Azure/aks-engine/blob/v0.55.0/examples/azure-stack/kubernetes-azurestack.json), [Windows](https://github.com/Azure/aks-engine/blob/v0.55.0/examples/azure-stack/kubernetes-windows.json)    |
|     v0.55.4    |     AKS Base Ubuntu 16.04-LTS Image Distro, September 2020   (2020.09.14), AKS Base Windows Image (17763.1397.200820)    |     1.15.12, 1.16.14, 1.17.11    |     [Linux](https://raw.githubusercontent.com/Azure/aks-engine/patch-release-v0.60.1/examples/azure-stack/kubernetes-azurestack.json), [Windows](https://raw.githubusercontent.com/Azure/aks-engine/patch-release-v0.60.1/examples/azure-stack/kubernetes-windows.json)    |
|     V0.60.1    |     AKS Base Ubuntu 16.04-LTS Image Distro, January 2021 (2021.01.28),   <br>AKS Base Ubuntu 18.04-LTS Image Distro, 2021 Q1 (2021.01.28), <br>AKS   Base Windows Image (17763.1697.210129)    |     1.16.14, 1.16.15, 1.17.17, 1.18.15    |     [Linux](https://raw.githubusercontent.com/Azure/aks-engine/patch-release-v0.60.1/examples/azure-stack/kubernetes-azurestack.json), [Windows](https://raw.githubusercontent.com/Azure/aks-engine/patch-release-v0.60.1/examples/azure-stack/kubernetes-windows.json)    |
    
<br></br>

3. In order to simplify the AKS Engine deployment process (i.e. Networking, DNS, Security), best practice is to create a jump box on your ASH instance that you can use for running AKS Engine. This jump box can be either Linux or Windows, and creation steps are available at the following:

    >Note: Before using these guides, make sure you know what AKS Engine version you plan to use, as you'll need that version number during the setup.
   
   * [Create a Linux Client](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux?view=azs-2008)
   * [Create a Windows Client](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows?view=azs-2008)

4. Finally, for our walk through we're making the assumption that you want to deploy to an existing Virtual Network in your Azure Stack Hub. By default AKS Engine will create a Vnet for you if you don't specify one, but it's rare to see that done in production deployments. In this Vnet you'll need a subnet for your control plane (master nodes) and a subnet for your worker nodes. The size will vary based on the network plugin you choose:

    * Kubenet(default): This network plugin provides an overlay network within the cluster, where an address space is assigned to a bridge network and all traffic in and out of the cluster goes through NAT to the individual node IP. IP space allocated should be at 1 IP per node, plus 1 IP for the cluster load balancer. 
    * Azure CNI: This network plugin allows the cluster to assign IP to pods using the subnet the cluster is deployed into via a virtual ethernet device in transparent mode. In this configuration you'll have 1 IP for each node plus an IP for each pod on a node (default: 30 pods per node), plus 1 IP for the cluster load balancer. 

    More details available [here](https://docs.microsoft.com/en-us/azure/aks/concepts-network)

---

## Time to deploy the cluster!

