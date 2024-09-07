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
```
k create deployment nginx --replicas=1 --image=nginx:1.23.4 --dry-run=client -o yaml > deployment.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
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
k apply -f deployment.yaml
```
2. Create a HorizontalPodAutoscaler for the Deployment named `nginx-hpa` that scales to minium of 3 and a maximum of 8 replicas. Scaling should happen based on an average CPU utilization of 75%, and an average memory utilization of 60%.
[HorizontalPodAutoscaler Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
```
kubectl autoscale deployment nginx --name=nginx-hpa --min=3 --max=8 --cpu-percent=75 --dry-run=client -o yaml > nginx-hpa.yaml
```
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  maxReplicas: 8
  minReplicas: 3
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
```
k apply -f nginx-hpa.yaml
```
3. Inspect the HorizontalPodAutoscaler object and identify the currently-utilized resources. How many replicas do you expect to exist?
```
k get hpa
NAME        REFERENCE          TARGETS                       MINPODS   MAXPODS   REPLICAS   AGE
nginx-hpa   Deployment/nginx   cpu: 0%/75%, memory: 1%/60%   3         8         3          45s
```

Cleanup
```
k delete -f nginx-hpa.yaml
k delete -f deployment.yaml
```
