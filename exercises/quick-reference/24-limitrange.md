```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
  namespace: d92
spec:
  limits:
  - min:
      cpu: 200m
    max:
      cpu: 500m
    type: Containe
```
