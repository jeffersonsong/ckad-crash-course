```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx:1.21.6
    name: nginx
    securityContext:
      readOnlyRootFilesystem: true
    volumeMounts:
    - mountPath: /var/run
      name: nginx-run
  volumes:
  - name: nginx-run
    emptyDir: {}
```
