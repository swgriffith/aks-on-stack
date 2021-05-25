# Pod Scaling

Pod scaling is something you can do manually using the ```kubectl scale deployment``` command, but likely you'll prefer the autoscaling provided by the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/). The HPA will scale a Kubenretes deployment out or back in based on metrics like CPU and Memory. If you need more advanced metrics, you can look at a solution like Microsoft's [Keda (Kubernetes Event Driven Autoscaler)](https://keda.sh/).

Let's use the HPA to auto-scale the sample application we deployed previously. We can do this either via kubectl or via a manifest file. We'll set up our UI to autoscale based on the pod CPU hitting 50 percent with a minimum pod count of 3 and a maximium of 10.

ex. Using kubectl
```bash
kubectl autoscale deployment service-tracker-ui -n service-tracker --cpu-percent=50 --min=3 --max=10

kubectl get hpa -n service-tracker

NAME                 REFERENCE                       TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
service-tracker-ui   Deployment/service-tracker-ui   0%/50%    3         10        10         8m31s
```

If using a manifest file, create a file containing the following yaml (ex. service-tracker-ui-hpa.yaml)
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: service-tracker-ui-hpa
spec:
  maxReplicas: 10 # define max replica count
  minReplicas: 3  # define min replica count
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: service-tracker-ui
  targetCPUUtilizationPercentage: 50 # target CPU utilization
```

```bash
# Apply the manifest
kubectl apply -f service-tracker-ui-hpa.yaml -n service-tracker

# Check the HPA
kubectl get hpa -n service-tracker

NAME                     REFERENCE                       TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
service-tracker-ui-hpa   Deployment/service-tracker-ui   1%/50%    3         10        3          75s
```

We can test our scaling by opening up two terminal windows. In one we'll watch the pods and in the other we'll run some test traffic against the service-tracker-ui. We're going to use [hey](https://github.com/rakyll/hey) to send the traffic, but you can use whatever tool you prefer.

```bash
# In terminal 1
watch kubectl top pod -n service-tracker
```

```bash
# In terminal 2
hey -z 5m --disable-keepalive http://172.23.112.58:8080/#/dashboard
```

Keep an eye on the 'watch' terminal window, and in a minute or so you should see the pod count jump up to 10. You can also take a look at the HPA to see whats going on. Notice that even with 10 pods our targets are getting hit with 122% CPU, so we could increase the max pods if needed.

```bash
kubectl get hpa -n service-tracker

NAME                     REFERENCE                       TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
service-tracker-ui-hpa   Deployment/service-tracker-ui   122%/50%   3         10        10         10m
```

---

* [Back to Scaling](./scaling.md)
* [Table of Contents](./README.md)