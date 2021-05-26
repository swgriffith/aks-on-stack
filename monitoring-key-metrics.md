# Monitoring Key Logs and Metrics

The following are some of the key logs and metrics to watch in your cluster. 

## Logs

While AKS in Azure Public Cloud provides a mechanism for pushing these logs out to solutions like LogAnalytics and EventHubs, that doesn't exist for AKS Engine on ASH. This is where a third party tool (ex. DataDog). AKS Engine does have a mechanism to export the logs, which you could then load into your prefered data store for query:

* [AKS Engine Log Export](https://github.com/Azure/aks-engine/blob/master/docs/topics/get-logs.md)

|Coponent|Description|Source|
|----|----|----|
|API-Server|The purpose of capturing these diagnostic logs is to be able to help troubleshoot problems on the api-server given it is a managed service. Some of the common problems highlighted in these logs are Admission Control pipeline errors such as a webhook blocking a deployment.|```kubectl logs -l component=kube-apiserver -n kube-system -f```|
|Cloud Controller|The purpose of capturing these diagnostic logs is to gain deeper visibility of issues that may arise between Kubernetes and the Azure control plane. A typical example is the AKS cluster having a lack of permissions to interact with Azure in some manner.|``` kubectl logs -l component=kube-controller-manager -n kube-system -f```<br/><br/>On Master:  ```sudo journalctl -u kubelet```|
|Audit logs|Send to Azure Storage. The purpose of capturing these diagnostic logs is usually for regulation purposes. These logs are quite large and can consume a lot of resources. If an organization does require these, and to keep costs to a minimum, the recommendation is to dump them straight to storage. This means they are not directly indexed, but can easily be pulled into tools if the need arises.|On Master: ```/var/log/kubeaudit``` |

## Metrics
|Name|Objective|Metric Name and Source|
|----|---------|----------------------|
|CPU, Memory and Network Utilization and Saturation|Provides key metrics into Node health|cadvisor reported data|
|Disk Space Utilization|Shows disk space usage which is important as workloads might not be able to pull down new container images and deploy for example.|Prometheus metric: ```node_filesystem_avail_bytes```|
|Disk I/O Spikes|Help identity spike patterns in disk usage. Word of caution, these metrics are averaged over a period of time so actual disk I/O usage spikes may be hidden. Please follow the iotop guidance provided.|Prometheus metric: ```node_disk_written_bytes_total```|
|Disk I/O Saturation|Help identity spike patterns in disk usage. Word of caution, these metrics are averaged over a period of time so actual disk I/O usage spikes may be hidden. Please follow the iotop guidance provided.|Prometheus metric:  ```node_disk_io_time_weighted_seconds```|
|API Server Total Requests|Metric that shows total requests to the api-server and rate over time (operations per second - ops) helps to see if there is a spike or increase in the load. For a frame of reference, a 3 node cluster with 4 CPU per node, with little to no workloads will be ~3 ops.A 3 node cluster 4 CPU per node that is fully loaded can be around ~25 ops. As you can see the number of requests can vary as it is highly dependent on the workloads running.|Prometheus metric: ```apiserver_request_total ```|
|API Server Requests Duration |Metric used to check how long api-server requests are taking which can be an indicator of things getting back up. |Prometheus metric: ```apiserver_request_duration_seconds_bucket ```|
|Kubelet errors|Metric that says whether kubelet is generating errors or not. Note, some errors are expected here and there, it is when there is a sustained growth of errors that it can highlight a potential problem about to happen. |Prometheus metric: ```kubelet_runtime_operations_errors_total```|
|Pod and Deployment Status|Monitor Pods and Deployments status of any not running Pods (Pending might indicate overcommitted nodes).|Prometheus metrics/Dashboard|
|CPU Requests vs CPU Requests% |Have we allocated enough resources to workloads. |kubectl top pod<br/>Prometheus metrics/Dashboard|
|Memory Requests vs Memory Requests%|Have we allocated enough resources to workloads. |kubectl top pod<br/>Prometheus metrics/Dashboard|
|ReplicaSet Desired vs Current Pods |Metrics that can be used to determine if the current state of Pods equals the desired state. Note, when cluster upgrades and nodes restarting are happening this might drop. This does not mean a bad thing as this is expected. The goal here is to be able to monitor this over a period of time and make sure the two states are not out of sync for an extended period.|Prometheus metrics: ```kube_deployment_spec_replicas kube_deployment_status_replicas ```|









