[Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello
spec:
  containers:
  - image: bmuschko/nodejs-hello-world:1.0.0
    name: hello
    ports:
    - name: nodejs-port
      containerPort: 3000
    readinessProbe:
      httpGet:
        port: nodejs-port
        path: /
      initialDelaySeconds: 2
    livenessProbe:
      httpGet:
        port: nodejs-port
        path: /
      initialDelaySeconds: 5
      periodSeconds: 8
```
