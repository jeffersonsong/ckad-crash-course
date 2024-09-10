[Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-security-context
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - image: busybox:1.28
    name: busybox-security-context
    args:
    - sh
    - -c
    - sleep 1h
    volumeMounts:
    - mountPath: /data/test
      name: data-volume
    securityContext:
      allowPrivilegeEscalation: false
  volumes:
  - name: data-volume
    emptyDir: {}

