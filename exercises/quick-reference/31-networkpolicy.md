[Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: network-policy
  namespace: k2
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: k1
    ports:
    - protocol: TCP
      port: 80
```

