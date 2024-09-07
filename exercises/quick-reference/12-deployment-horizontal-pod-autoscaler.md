Create a HorizontalPodAutoscaler for the Deployment named `nginx-hpa` that scales to minium of 3 and a maximum of 8 replicas. Scaling should happen based on an average CPU utilization of 75%, and an average memory utilization of 60%.

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
