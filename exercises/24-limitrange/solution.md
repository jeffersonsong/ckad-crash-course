# Exercise 24

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `d92`<br>
* Documentation: [Limit Ranges](https://kubernetes.io/docs/concepts/policy/limit-range/), [Resource Management for Pods and Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

</p>
</details>

A LimitRange can restrict resource consumption for Pods in a namespace, and assign default computing resource if no resource requirements have been defined. You will practice the effects of a LimitRange on the creation of a Pod in different scenarios.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Creating a Pod Conforming with LimitRange in a Namespace"](https://learning.oreilly.com/scenarios/creating-a-pod/9781098164201/).

1. Inspect the YAML manifest definition in the file [`setup.yaml`](./setup.yaml).

```
k apply -f setup.yaml 
namespace/d92 created
limitrange/cpu-limit-range created
```

2. Create the objects from the YAML manifest file.
3. Create a new Pod named `pod-without-resource-requirements` in the namespace `d92` that uses the container image `nginx:1.23.4-alpine` without any resource requirements. Inspect the Pod details. What resource definitions do you expect to be assigned?
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-without-resource-requirements
  namespace: d92
spec:
  containers:
  - image: nginx:1.23.4-alpine
    name: pod-without-resource-requirements
```
Use default cpu: 500m

```
k apply -f pod-without-resource-requirements.yaml

k describe pod pod-without-resource-requirements -n d92

Containers:
  pod-without-resource-requirements:
    Limits:
      cpu:  500m
    Requests:
      cpu:        500m
```

4. Create a new Pod named `pod-with-more-cpu-resource-requirements` in the namespace `d92` that uses the container image `nginx:1.23.4-alpine` with a CPU resource request of 400m and limits of 1.5. What runtime behavior do you expect to see?

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-more-cpu-resource-requirements
  namespace: d92
spec:
  containers:
  - image: nginx:1.23.4-alpine
    name: pod-with-more-cpu-resource-requirements
    resources:
      requests:
        cpu: 400m
      limits:
        cpu: "1.5"
```
```
k apply -f pod-with-more-cpu-resource-requirements.yaml
Error from server (Forbidden): error when creating "pod-with-more-cpu-resource-requirements.yaml": pods "pod-with-more-cpu-resource-requirements" is forbidden: maximum cpu usage per Container is 500m, but limit is 1500m
```

5. Create a new Pod named `pod-with-less-cpu-resource-requirements` in the namespace `d92` that uses the container image `nginx:1.23.4-alpine` with a CPU resource request of 350m and limits of 400m. What runtime behavior do you expect to see?

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-less-cpu-resource-requirements
  namespace: d92
spec:
  containers:
  - image: nginx:1.23.4-alpine
    name: pod-with-less-cpu-resource-requirements
    resources:
      requests:
        cpu: 350m
      limits:
        cpu: 400m
```
```
k apply -f pod-with-less-cpu-resource-requirements.yaml    
pod/pod-with-less-cpu-resource-requirements created

k describe pod pod-with-less-cpu-resource-requirements -n d92
Containers:
  pod-with-less-cpu-resource-requirements:
    Limits:
      cpu:  400m
    Requests:
      cpu:        350m
```


Cleanup
```
k delete pod pod-with-less-cpu-resource-requirements -n d92
k delete pod pod-without-resource-requirements -n d92
k delete limitrange cpu-limit-range -n d92
k delete ns d92
```