[Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend
spec:
  containers:
  - image: nginx:1.23.4-alpine
    name: backend
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
```
