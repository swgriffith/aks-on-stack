
# Cluster Scaling
As for cluster scaling, unfortunately, Azure Stack Hub doesn't support Virtual Machine Scale Sets, which are used by the cluster autoscaler. Any nodepool scaling operations will need to be done manually, or through a custom scripting approach.
AKS Engine provides an ```aks-engine scale``` command for scaling operations. To run the scale command you just need to provide the following values:

* **azure-env:** AzureStackCloud
* **location:** The location id of your instance
* **resource-group:** The resource group name where your cluster is deployed
* **api-model:** This is the path to the apimodel file that was created as an output of your cluster create (**Note:** dont mix this up with the file you created as an input to your cluster creation or the command will fail)
* **client-id:** The client ID you use for cluster operations
* **client-secret:** The secret for the client ID use for cluster operations
* **subscription-id:** ID for the subscription containing the cluster
* **new-node-count:** Target number of nodes in the cluster nodepool
* **node-pool:** Name of the nodepool you want to scale
* **apiserver:** FQDN of the cluster API server, used to execute the drain and cordon commands when scaling the cluster

ex.
```bash
aks-engine scale \
--azure-env AzureStackCloud \
--location 3173r03a \
--resource-group griffith-cni-aks \
--api-model ./griffith-cni-aks/apimodel.json \
--client-id "<Insert Client ID>" \
--client-secret "<Insert Client Secret" \
--subscription-id "<Insert Subscription ID>" \
--new-node-count 2 \
--node-pool linuxpool \
--apiserver "cni-aks-ash.3173r03a.cloudapp.azcatcpec.com"
```

---

* [Back to Scaling](./scaling.md)
* [Table of Contents](./README.md)