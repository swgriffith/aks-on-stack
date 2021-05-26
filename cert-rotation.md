# Certificate Rotation

Kubernetes relies HEAVILY on certificates for most components of the platform. When you create a cluster with AKS Engine, all of those certificates are generated for you and place in the output directory. If you have a need to rotate those certificates, either because they're soon to expire, or there's been a security breach, or just for general security best practice, AKS Engine provides a mechanism. 

>**_NOTE:_** The AKS Engine on ASH document provides a very detailed overview [here](https://docs.microsoft.com/en-us/azure-stack/user/kubernetes-aks-engine-rotate-certs?view=azs-2102). I highly recommend you read thorugh this to fully understand your options.

First, just to show you your active certificates and their expirations, you can use the following:
```bash
# Navigate to your output directory containing all of your key and crt files
cd <aks-engine output directory>

# Loop through the files and show the relevant dates
for i in *.crt; do echo $i  $(cat $i|openssl x509 -noout -dates); done

apiserver.crt notBefore=May 24 17:23:18 2021 GMT notAfter=May 17 17:23:18 2051 GMT
ca.crt notBefore=May 24 17:23:16 2021 GMT notAfter=May 17 17:23:16 2051 GMT
client.crt notBefore=May 24 17:23:18 2021 GMT notAfter=May 17 17:23:18 2051 GMT
etcdclient.crt notBefore=May 24 17:23:18 2021 GMT notAfter=May 17 17:23:18 2051 GMT
etcdpeer0.crt notBefore=May 24 17:23:18 2021 GMT notAfter=May 17 17:23:18 2051 GMT
etcdpeer1.crt notBefore=May 24 17:23:18 2021 GMT notAfter=May 17 17:23:18 2051 GMT
etcdpeer2.crt notBefore=May 24 17:23:18 2021 GMT notAfter=May 17 17:23:18 2051 GMT
etcdserver.crt notBefore=May 24 17:23:18 2021 GMT notAfter=May 17 17:23:18 2051 GMT
kubectlClient.crt notBefore=May 24 17:23:18 2021 GMT notAfter=May 17 17:23:18 2051 GMT
```

As you can see from the above, there's no rush to rotate these certs, since they don't expire until 2051, but maybe something was leaked, or we're just trying to be diligent with cluster security.

Let's rotate the certificates:

```bash
# Rotate Certs
# Obviously update with your correct paths and API Server FQDN
aks-engine rotate-certs \
--azure-env AzureStackCloud \
--location 3173r03a \
--resource-group griffith-cni-aks \
--api-model ./griffith-cni-aks/apimodel.json \
--client-id "<Insert Client ID>" \
--client-secret "<Insert Client Secret" \
--subscription-id "<Insert Subscription ID>" \
--linux-ssh-private-key ~/.ssh/id_rsa \
--ssh-host "cni-aks-ash.3173r03a.cloudapp.azcatcpec.com"
```

After you run this command, your certs will begin to rotate and you will lose access to the API Server as nodes are updated and rebooted. This is a control plane change, so your application workloads should continue to function.

The error you may see if you try to access the API server during this time is as follows:

```bash
kubectl get nodes

Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "ca")
```

Once the command is complete, you'll have an output directory named ```_rotate_certs_output```. This directory contains all of your new certs **as well as your new kubeconfig** which you'll need to use for kubectl commands going forward.

---
* [Table of Contents](./README.md)