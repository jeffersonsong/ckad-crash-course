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

[list or watch objects of kind Servic](https://kubernetes.io/docs/reference/kubernetes-api/service-resources/service-v1/#list-list-or-watch-objects-of-kind-service)
```
HTTP Request
GET /api/v1/namespaces/{namespace}/services
```
[DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
```
<namespace>.svc.cluster.local
```

3. Create and attach the service account `api-call` to the Pod.

[Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

```
k create sa api-call -n t23

k replace --force -f service-list.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: service-list
  name: service-list
  namespace: t23
spec:
  serviceAccountName: api-call
  containers:
  - image: alpine/curl:3.14
    name: service-list
    command:
    - /bin/sh
    - -c
    - 'while true; do curl -s -k -m 5 -H "Authorization: Bearer $(cat /var/run/kubernetes.io/serviceaccount/token)" https://kubernetes.default.svc.cluster.local/api/v1/namespaces/default/services; sleep 10; done'
```

```
k apply -f service-list.yaml

k get pod -n t23
NAME           READY   STATUS    RESTARTS   AGE
service-list   1/1     Running   0          31s
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
k create clusterrole -h | less


k create clusterrole list-services-clusterrole --verb=list --resource=services

clusterrole.rbac.authorization.k8s.io/list-services-clusterrole created

k create rolebinding -h | less

k create rolebinding api-call-list-services-rolebinding --clusterrole=list-services-clusterrole --serviceaccount=t23:api-call                        
rolebinding.rbac.authorization.k8s.io/api-call-list-services-rolebinding created
```
