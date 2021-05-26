# Enable Monitoring

When running an AKS cluster in Azure you have the ability to quickly and easily attach that cluster to a Log Analytics workspace and enable Container Insights. Since Log Analytics isnt available as a resource in Azure Stack Hub, you can still use Container Insights, but you need to point it at a Log Analytics instance in public Azure.

Alternatively, or in addition to Container Insights, you can run a logging solution within the cluster. The most popular and common in-cluster monitoring solution is commonly refered to as 'Prometheus'. I say commonly referred to because Prometheus itself is just a data store and not a complete monitoring solution. You need other components to come together to get a full monitoring solution. This typically includes Alert Manager, Grafana and a series of exporters used to load monitoring data into Prometheus. To simplify the setup of these components, you can look at projects like the [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator) and [Kube Prometheus](https://github.com/prometheus-operator/kube-prometheus).

* [Container Insights](./container-insights.md)
* [Kube-Prometheus](./prometheus.md)


---

* [Table of Contents](./README.md)