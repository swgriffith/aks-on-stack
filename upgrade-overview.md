# Cluster Upgrades - Overview

When talking about upgrades on your AKS Engine based application hosting platform, there are several layers that need to be considered. Lets explore from the top down:


|Layer|Description|
|-----|-----------|
|Application|As you're rolling new versions of your application you'll obviously need to have a process in place to upgrade the application version. This typically involves a CI/CD pipeline that continually builds new conatiner images and pushes changes to the hosting environment as the result of some trigger (ex. Git Push or Pull Request)|
|Kubernetes Version|The Kubernetes open source project is relatively young, so new features are contantly coming out, as well as new bug fixes and CVE's being addressed. Keeping up to date on Kubernetes versions is critical to success on the platform.|
|OS Version|Any OS will constantly have bug fixes and CVEs being addressed via patches and upgrades. While AKS Engine does implement a solution for automatic patching of the OS, you still need to be in control of when those go into effect, in particular if a reboot is required.|
|AKS Engine|AKS Engine itself is an active open source project with regular updates. These updates are used to handle node OS image updates, Azure capability improvements as well as various other updates that materially impact the overall cluster deployment as compared to more soft updates like those noted above.|

For the purposes of this guide we'll leave Application versioning out of the conversation and focus on the lower layers. Fortunately, AKS Engine provides almost all of the tools we need to keep up to date. We just need to implement a plan and schedule to update.

### AKS Engine

AKS Engine is a good starting point, as it's the foundation of our whole cluster. AKS Engine goes through regular releases which can be tracked at the [AKS Engine Releases](https://github.com/Azure/aks-engine/releases) page. If you want to keep up to date on these you can subscribe to notifications via [GitHub Notifications](https://docs.github.com/en/github/managing-subscriptions-and-notifications-on-github/setting-up-notifications/about-notifications).

While keeping current on AKS Engine updates is important, for Azure Stack Hub we need to keep an eye on the releases that have been verified by the Azure Stack Hub team. There currently isn't a notification you can subscribe to for these updates, however you can track the current supported versions in the AKS Release Notes in the ASH documentation [here](https://docs.microsoft.com/en-us/azure-stack/user/kubernetes-aks-engine-release-notes?view=azs-2102#aks-engine-and-azure-stack-version-mapping).

To update your AKS Engine version you simply run the following:
```bash
# First, if you havent already downloaded the install script
curl -o get-akse.sh https://raw.githubusercontent.com/Azure/aks-engine/master/scripts/get-akse.sh
chmod 700 get-akse.sh

# Next, you run the command and specify the latest version supported on your Azure Stack Hub
./get-akse.sh --version v0.xx.x

```

>**_NOTE:_** When you change aks-engine versions, often the apimodel will change. If you get an 'error parsing apimodel' message when trying to run aks-engine commands, double check your apimodel file matches your aks-engine version. There are example files for aks-engine on Azure Stack Hub [here](https://github.com/Azure/aks-engine/tree/release-v0.60.0/examples/azure-stack). You can change the branch to select the version you're targetting.

### OS Version & Patching

For the OS Version (i.e. Ubuntu 16.04 vs. 18.04) you get those updates via AKS Engine itself. When AKS Engine officially drops support, or adds support, for an OS Version you will see that included in the release notes. You then have the ability to specify that OS version in your apimodel.json. 

Patches are applied to your AKS Engine nodes automatically via the [Unattended Upgrade](https://wiki.debian.org/UnattendedUpgrades) capability of the OS. Unattended Upgrades will ensure that CVEs are patched, however it **WILL NOT** reboot your nodes to make those updates effective. There are a few ways you can address this:

### Manual Drain and Cordon

Create a regularly scheduled process where you execute a cluster reboot yourself. To do this you would want to use the Kuberntes [Drain and Cordon](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/) process. When you drain a node it gets marked as 'Not Schedulable' and the pods are removed from the node. The replica set for the deployment will make sure new pods are started on other nodes. Once drained, you can reboot the node. Once back online you can 'uncordon' the node to make it schedulable again.

```bash
# Drain a node
kubectl get nodes
kubectl drain <node name>

# Once drained, you reboot
# Watch for the node to be back online
watch kubectl get nodes

# Once in Ready,Not Schedulable you can uncordon
kubectl uncordon <node name>
```

### Use **KU**bernetes **RE**boot **D**aemon (aka Kured)

As you can imagine after reading the manual process above, it's a heavily manual process going node by node, so you'd likely automate. Fortunately, someone has already created a tool that automates this process. [Kured](https://github.com/weaveworks/kured#readme). 

Kured is a daemonset that runs in your cluster that allows you to configure how and when you want reboots to take place. You can set it up to reboot on a schedule, or you can tell it to reboot when it notices that an Unattended Upgrade has occured that requires a reboot. 

NEXT: [Upgrade Walkthrough](./upgrade-walkthrough.md)

---
* [Table of Contents](./README.md)