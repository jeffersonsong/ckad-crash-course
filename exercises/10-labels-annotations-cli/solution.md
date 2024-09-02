# Exercise 10

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/), [Annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/)

</p>
</details>

In this exercise, you will exercise assigning labels and annotations to a set of Pods. Moreover, you will use `kubectl` to query for Pods based on different requirements.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive labs ["Assigning Labels to Pods Imperatively"](https://learning.oreilly.com/scenarios/assigning-labels-to/9781098163952/) and ["Assigning Annotations to Pods Imperatively"](https://learning.oreilly.com/scenarios/assigning-annotations-to/9781098163976/).

1. Create three different Pods with the names `frontend`, `backend` and `database` that use the image `nginx:1.25.5-alpine`. For convenience, you can use the file [`pods.yaml`](./pods.yaml) to create the Pods.

k apply -f pods.yaml

2. Declare labels for those Pods, as follows:

- `frontend`: `env=prod`, `team=shiny`
- `backend`: `env=prod`, `team=legacy`, `app=v1.2.4`
- `database`: `env=prod`, `team=storage`

```shell
k label -h

k label pod frontend env=prod team=shiny
pod/frontend labeled

k label pod backend env=prod team=legacy app=v1.2.4
pod/backend labeled

k label pod database env=prod team=storage
pod/database labeled
```

3. Declare annotations for those Pods, as follows:

- `frontend`: `contact=John Doe`, `commit=2d3mg3`
- `backend`: `contact=Mary Harris`

```shell
k annotate pod -h

k annotate pod frontend contact='John Doe' commit=2d3mg3
k annotate pod backend contact='Mary Harris'
```

4. Render the list of all Pods and their labels.

```shell
k get pod --show-labels
NAME       READY   STATUS    RESTARTS   AGE     LABELS
backend    1/1     Running   0          2m57s   app=v1.2.4,env=prod,team=legacy
database   1/1     Running   0          2m57s   env=prod,team=storage
frontend   1/1     Running   0          2m57s   env=prod,team=shiny
```

5. Use label selectors on the command line to query for all production Pods that belong to the teams `shiny` and `legacy`.

```shell
k get pods -l 'team in (shiny, legacy)',env=prod --show-labels
NAME       READY   STATUS    RESTARTS   AGE     LABELS
frontend   1/1     Running   0          9m59s   env=prod,team=shiny
backend   1/1     Running   0          10m   app=v1.2.4,env=prod,team=legacy
```

6. Remove the label `env` from the `backend` Pod and rerun the selection.

```shell
k label pod -h

kubectl label pod backend env-
pod/backend unlabeled
```

7. Render the surrounding 3 lines of YAML of all Pods that have annotations.

```shell
k get pod -o yaml | grep -C 3 "annotations:"
- apiVersion: v1
  kind: Pod
  metadata:
    annotations:
      contact: Mary Harris
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"backend","namespace":"default"},"spec":{"containers":[{"image":"nginx:1.25.5-alpine","name":"nginx"}],"restartPolicy":"Never"}}
--
- apiVersion: v1
  kind: Pod
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"database","namespace":"default"},"spec":{"containers":[{"image":"nginx:1.25.5-alpine","name":"nginx"}],"restartPolicy":"Never"}}
    creationTimestamp: "2024-09-02T02:14:33Z"
--
- apiVersion: v1
  kind: Pod
  metadata:
    annotations:
      commit: 2d3mg3
      contact: John Doe
      kubectl.kubernetes.io/last-applied-configuration: |
```
