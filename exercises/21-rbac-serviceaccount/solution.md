# Exercise 21

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `t23`<br>
* Documentation: [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/), [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

</p>
</details>

In this exercise, you will define Role Based Access Control (RBAC) to grant permissions to a service account. The permissions should only apply to certain API resources and operations.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Regulating Access to API Resources with RBAC"](https://learning.oreilly.com/scenarios/regulating-access-to/9781098164171/).
1. Create a new namespace named `t23`.
```
k create ns t23
```
2. Create a Pod named `service-list` in the namespace `t23`. The container uses the image `alpine/curl:3.14` and makes a `curl` call to the Kubernetes API that lists Service objects in the `default` namespace in an infinite loop.

Search Kubernetes API lists Service
[Kubernetes API Concepts](https://kubernetes.io/docs/reference/using-api/api-concepts/)
[DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
[Accessing the Kubernetes API from a Pod](https://kubernetes.io/docs/tasks/run-application/access-api-from-pod/)
[Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
kubernetes.default.svc.cluster.local/api/v1/namespaces/default/services
kubernetes.default.svc/api/v1/namespaces/default/services

TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)


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

3. Create and attach the service account `api-call` to the Pod.
```
k create sa api-call -n t23
k apply -f pod.yaml
```
4. Inspect the container logs after the Pod has been started. What response do you expect to see from the `curl` command?
```
k logs service-list -n t23
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "services is forbidden: User \"system:serviceaccount:t23:api-call\" cannot list resource \"services\" in API group \"\" in the namespace \"default\"",
  "reason": "Forbidden",
  "details": {
    "kind": "services"
  },
  "code": 403
}
```

5. Assign a ClusterRole and RoleBinding to the service account that only allows the operation needed by the Pod. Have a look at the response from the `curl` command.
```
kubectl create clusterrole service-list-clusterrole --verb=list --resource=services

kubectl create rolebinding api-call-rolebinding --clusterrole=service-list-clusterrole --serviceaccount=t23:api-call

k logs service-list -n t23
```

Cleanup
```
k delete rolebinding api-call-rolebinding
k delete clusterrole service-list-clusterrole
k delete pod service-list -n t23
k delete ns t23
```
