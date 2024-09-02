# Exercise 11

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/), [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)

</p>
</details>

In this exercise, you will create a Deployment with multiple replicas. After inspecting the Deployment, you will update its Pod template. Furthermore, you will use the rollout history to roll back to a previous revision.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive labs ["Creating and Manually Scaling a Deployment"](https://learning.oreilly.com/scenarios/creating-and-manually/9781098164010/) and ["Rolling Out a New Revision for a Deployment"](https://learning.oreilly.com/scenarios/rolling-out-a/9781098164027/).

1. Create a Deployment named `nginx` with 3 replicas. The Pods should use the `nginx:1.23.0` image and the name `nginx`. The Deployment uses the label `tier=backend`. The Pod template should use the label `app=v1`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    tier: backend
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: v1
  template:
    metadata:
      labels:
        app: v1
    spec:
      containers:
      - image: nginx:1.23.0
        name: nginx
```
```shell
k apply -f deployment.yaml
```

2. List the Deployment and ensure that the correct number of replicas is running.

```shell
k get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           19s
```

3. Update the image to `nginx:1.23.4`.

```shell
k set image deployment/nginx nginx=nginx:1.23.4
deployment.apps/nginx image updated
```

4. Verify that the change has been rolled out to all replicas.

```shell
k get deploy -o wide
NAME    READY   UP-TO-DATE   AVAILABLE   AGE    CONTAINERS   IMAGES         SELECTOR
nginx   3/3     1            3           115s   nginx        nginx:1.23.4   app=v1
```

5. Assign the change cause "Pick up patch version" to the revision.

```shell
k rollout history deployment/nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

k annotate deployment/nginx kubernetes.io/change-cause="Pick up patch version"
deployment.apps/nginx annotated

```shell
k rollout history deployment/nginx                                            
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
1         <none>
2         Pick up patch version
```

6. Scale the Deployment to 5 replicas.

```shell
k scale deployment/nginx --replicas=5
```

7. Have a look at the Deployment rollout history.

```shell
k rollout history deploy/nginx   
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
1         <none>
2         Pick up patch version
```

8. Revert the Deployment to revision 1.

```shell
k rollout -h
kubectl rollout undo -h 
kubectl rollout undo deployment/nginx --to-revision=1
deployment.apps/nginx rolled back

k rollout history deploy/nginx             
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
2         Pick up patch version
3         <none>
```

9. Ensure that the Pods use the image `nginx:1.23.0`.

```shell
k get deploy -o wide
NAME    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
nginx   5/5     5            5           14m   nginx        nginx:1.23.0   app=v1
```

10. (Optional) Discuss: Can you foresee potential issues with a rolling deployment? How do you configure a update process that first kills all existing containers with the current version before it starts containers with the new version?

```yaml
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
```

```yaml
  strategy:
    type: Recreate
```
