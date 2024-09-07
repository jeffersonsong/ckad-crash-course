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
```
k run frontend --image=nginx:1.25.5-alpine --restart=Never
k run backend --image=nginx:1.25.5-alpine --restart=Never
k run database --image=nginx:1.25.5-alpine --restart=Never
```
2. Declare labels for those Pods, as follows:

- `frontend`: `env=prod`, `team=shiny`
- `backend`: `env=prod`, `team=legacy`, `app=v1.2.4`
- `database`: `env=prod`, `team=storage`
```
k label pods frontend env=prod team=shiny
k label pods backend env=prod team=legacy app=v1.2.4
k label pods database env=prod team=storage
```

3. Declare annotations for those Pods, as follows:

- `frontend`: `contact=John Doe`, `commit=2d3mg3`
- `backend`: `contact=Mary Harris`

```
k annotate pods frontend contact='John Doe' commit=2d3mg3
k annotate pods backend contact='Mary Harris'
```

4. Render the list of all Pods and their labels.
```
k get pods --show-labels
NAME       READY   STATUS    RESTARTS   AGE     LABELS
backend    1/1     Running   0          5m59s   app=v1.2.4,env=prod,run=backend,team=legacy
database   1/1     Running   0          5m45s   env=prod,run=database,team=storage
frontend   1/1     Running   0          6m38s   env=prod,run=frontend,team=shiny
```
5. Use label selectors on the command line to query for all production Pods that belong to the teams `shiny` and `legacy`.
```
k get pods -l 'team in (shiny, legacy)'  
NAME       READY   STATUS    RESTARTS   AGE
backend    1/1     Running   0          11m
frontend   1/1     Running   0          12m
```
6. Remove the label `env` from the `backend` Pod and rerun the selection.
```
k label pod database env-
pod/database unlabeled
```
7. Render the surrounding 3 lines of YAML of all Pods that have annotations.
```
k get pod -o yaml | grep -C 3 "annotations"
- apiVersion: v1
  kind: Pod
  metadata:
    annotations:
      contact: Mary Harris
    creationTimestamp: "2024-09-07T13:40:26Z"
    labels:
--
- apiVersion: v1
  kind: Pod
  metadata:
    annotations:
      commit: 2d3mg3
      contact: John Doe
    creationTimestamp: "2024-09-07T13:39:47Z"
```

Cleanup
```
k delete pods frontend backend database
```
