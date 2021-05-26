# AKS Engine on Azure Stack Hub - Best Practices

The purpose of this document is to provide a light weight guide to AKS on Azure Stack Hub deployment. The best, in depth, source for guidance is the [Azure Documentation](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-2008) which I used to create this summary content. Think of this content as the CliffsNotes™ version.

**Table of Contents:**
* [Getting Started](./getting-started.md)
  * [Cluster Creation](./deploy-cluster.md)
* [Monitoring](./monitoring.md)
  * [Container Insights](./container-insights.md)
  * [Prometheus](./prometheus.md)
* [Scaling](./scaling.md)
  * [Pod Scaling](./pod-scaling.md)
  * [Cluster Scaling](./cluster-scaling.md)
* [Upgrades](./upgrade-overview.md)
  * [Upgrade Walkthrough](./upgrade-walkthrough.md)
* [Certificate Rotation](./cert-rotation.md)