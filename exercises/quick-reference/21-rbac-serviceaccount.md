Create a Pod named `service-list` in the namespace `t23`. The container uses the image `alpine/curl:3.14` and makes a `curl` call to the Kubernetes API that lists Service objects in the `default` namespace in an infinite loop.

Search Kubernetes API lists Service
- [Kubernetes API Concepts](https://kubernetes.io/docs/reference/using-api/api-concepts/)
- [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
- [Accessing the Kubernetes API from a Pod](https://kubernetes.io/docs/tasks/run-application/access-api-from-pod/)
- [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
```
# DNS
kubernetes.default.svc.cluster.local
kubernetes.default.svc

# Endpoint
GET /api/v1/namespaces/default/services

# Token
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)

curl -s -k -H "Authorization: Bearer $TOKEN" https://kubernetes.default.svc.cluster.local/api/v1/namespaces/default/services
```

```
k run service-list -n t23 --image=alpine/curl:3.14 --dry-run=client -o yaml > pod.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: service-list
  namespace: t23
spec:
  containers:
  - image: alpine/curl:3.14
    name: service-list
    command:
    - /bin/sh
    - -c
    - 'TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token);while true; do curl -s -k -H "Authorization: Bearer $TOKEN" https://kubernetes.default.svc.cluster.local/api/v1/namespaces/default/services;sleep 10; done;'
  serviceAccountName: api-call
```

Create and attach the service account `api-call` to the Pod.
```
k create sa api-call -n t23
k apply -f pod.yaml
```

Assign a ClusterRole and RoleBinding to the service account that only allows the operation needed by the Pod. Have a look at the response from the `curl` command.
```
kubectl create clusterrole service-list-clusterrole --verb=list --resource=services
clusterrole.rbac.authorization.k8s.io/service-list-clusterrole created

kubectl create rolebinding api-call-rolebinding --clusterrole=service-list-clusterrole --serviceaccount=t23:api-call
```

