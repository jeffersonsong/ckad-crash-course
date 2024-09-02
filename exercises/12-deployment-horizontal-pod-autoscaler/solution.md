# Exercise 12

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)

</p>
</details>

> [!IMPORTANT]
> You will need to install the metrics server if you want actual resource metrics to be collected and displayed by the HorizontalPodAutoscaler. You can find [installation instructions](https://github.com/kubernetes-sigs/metrics-server#installation) on the project's GitHub page.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Creating a Horizontal Pod Autoscaler for a Deployment"](https://learning.oreilly.com/scenarios/creating-a-horizontal/9781098164034/).

1. Create a Deployment named `nginx:1.23.4` with 1 replica. The Pod template of the Deployment should use container image `nginx:1.23.4`, set the CPU resource request to 0.5, and the memory resource request/limit to 500Mi.

[Assign CPU Resources to Containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.23.4
        name: nginx
        resources: 
          requests:
            cpu: 0.5
            memory: 500Mi
          limits:
            memory: 500Mi
```

```
k apply -f nginx-deployment.yaml
k get deploy -o wide
NAME    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
nginx   1/1     1            1           92s   nginx        nginx:1.23.4   app=nginx
```

2. Create a HorizontalPodAutoscaler for the Deployment named `nginx-hpa` that scales to minium of 3 and a maximum of 8 replicas. Scaling should happen based on an average CPU utilization of 75%, and an average memory utilization of 60%.

[HorizontalPodAutoscaler Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
k autoscale -h

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  minReplicas: 3
  maxReplicas: 8
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 75
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 60
```
k apply -f nginx-hpa.yaml
horizontalpodautoscaler.autoscaling/nginx-hpa created
```

3. Inspect the HorizontalPodAutoscaler object and identify the currently-utilized resources. How many replicas do you expect to exist?

```
k get horizontalpodautoscaler nginx-hpa        
NAME        REFERENCE          TARGETS                       MINPODS   MAXPODS   REPLICAS   AGE
nginx-hpa   Deployment/nginx   cpu: 0%/75%, memory: 1%/60%   3         8         3          2m20s

k describe horizontalpodautoscaler nginx-hpa
Name:                                                     nginx-hpa
Namespace:                                                default
Labels:                                                   <none>
Annotations:                                              <none>
CreationTimestamp:                                        Mon, 02 Sep 2024 08:30:14 -0400
Reference:                                                Deployment/nginx
Metrics:                                                  ( current / target )
  resource cpu on pods  (as a percentage of request):     0% (0) / 75%
  resource memory on pods  (as a percentage of request):  1% (9984682666m) / 60%
Min replicas:                                             3
Max replicas:                                             8
Deployment pods:                                          3 current / 3 desired
Conditions:
  Type            Status  Reason               Message
  ----            ------  ------               -------
  AbleToScale     True    ScaleDownStabilized  recent recommendations were higher than current one, applying the highest recent recommendation
  ScalingActive   True    ValidMetricFound     the HPA was able to successfully calculate a replica count from memory resource utilization (percentage of request)
  ScalingLimited  False   DesiredWithinRange   the desired count is within the acceptable range
Events:
  Type    Reason             Age   From                       Message
  ----    ------             ----  ----                       -------
  Normal  SuccessfulRescale  64s   horizontal-pod-autoscaler  New size: 3; reason: Current number of replicas below Spec.MinReplicas
```