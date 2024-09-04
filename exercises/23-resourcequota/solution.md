# Exercise 23

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `rq-demo`<br>
* Documentation: [Resource Quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/)

</p>
</details>

In this exercise, you will create a ResourceQuota with specific CPU and memory limits for a new namespace. Pods created in the namespace will have to adhere to those limits.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Defining a Resource Quota for a Namespace"](https://learning.oreilly.com/scenarios/defining-a-resource/9781098164195/).

Create a resource quota named `app` under the namespace `rq-demo` using the following YAML definition in the file [`resourcequota.yaml`](./resourcequota.yaml).

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: app
spec:
  hard:
    pods: "2"
    requests.cpu: "2"
    requests.memory: 500Mi
```

```
k apply -f resourcequota.yaml -n rq-demo
resourcequota/app created
```

1. Create a new Pod that exceeds the limits of the resource quota requirements e.g. by defining 1Gi of memory but stays below the CPU e.g. 0.5. Write down the error message.

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
  namespace: rq-demo
spec:
  containers:
  - image: nginx
    name: nginx
    resources: 
      requests:
        cpu: 1
        memory: 1Gi
```

```
k apply -f pod.yaml
Error from server (Forbidden): error when creating "pod.yaml": pods "nginx" is forbidden: exceeded quota: app, requested: requests.memory=1Gi, used: requests.memory=0, limited: requests.memory=500Mi
```

2. Change the request limits to fulfill the requirements to ensure that the Pod could be created successfully. Write down the output of the command that renders the used amount of resources for the namespace.

```yaml
    resources:
      requests:
        cpu: 1
        memory: 500Mi
```

```
k apply -f pod.yaml
pod/nginx created
```
```
k describe resourcequota app -n rq-demo
Name:            app
Namespace:       default
Resource         Used   Hard
--------         ----   ----
pods             1      2
requests.cpu     1      2
requests.memory  500Mi  500Mi
```

Cleanup

```
k delete pod nginx -n rq-demo 

k delete resourcequota app -n rq-demo
```