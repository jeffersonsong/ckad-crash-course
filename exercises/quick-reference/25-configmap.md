[Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

```yaml
  volumes:
  - name: configpv
    configMap:
      name: app-config
```
