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
```
k create deploy nginx --replicas=3 --image=nginx:1.23.0 --dry-run=client -o yaml > nginx-deployment.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    tier: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: v1
  strategy: {}
  template:
    metadata:
      labels:
        app: v1
      name: nginx
    spec:
      containers:
      - image: nginx:1.23.0
        name: nginx
```
```
k apply -f nginx-deployment.yaml
```
2. List the Deployment and ensure that the correct number of replicas is running.
```
k get deploy nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           25s
```
3. Update the image to `nginx:1.23.4`.
```
k set image deploy/nginx nginx=nginx:1.23.4
deployment.apps/nginx image updated
```
4. Verify that the change has been rolled out to all replicas.
```
k describe deploy nginx

OldReplicaSets:  nginx-b97866f9 (0/0 replicas created)
NewReplicaSet:   nginx-74d8979fcf (3/3 replicas created)

kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

kubectl rollout history deployment nginx --revision=2
deployment.apps/nginx with revision #2
Pod Template:
  Labels:	app=v14
	pod-template-hash=5bd95c598
  Containers:
   nginx:
    Image:	nginx:1.23.4
    Port:	<none>
    Host Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```
5. Assign the change cause "Pick up patch version" to the revision.
[Update CHANGE-CAUSE rollout description for a specific revision](https://discuss.kubernetes.io/t/update-change-cause-rollout-description-for-a-specific-revision/18722)
```
k rollout history deploy/nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

k annotate deploy/nginx kubernetes.io/change-cause='Pick up patch version'
deployment.apps/nginx annotated

k rollout history deploy/nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
1         <none>
2         Pick up patch version
```
6. Scale the Deployment to 5 replicas.
```
k scale deploy/nginx --replicas=5
deployment.apps/nginx scaled
```
7. Have a look at the Deployment rollout history.
```
k rollout history deploy/nginx   
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
1         <none>
2         Pick up patch version
```
8. Revert the Deployment to revision 1.
```
kubectl rollout undo deploy/nginx --to-revision=1
deployment.apps/nginx rolled back

k rollout history deploy/nginx              
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
2         Pick up patch version
3         <none>
```
9. Ensure that the Pods use the image `nginx:1.23.0`.
```
k describe pod nginx-b97866f9-7phc2 | grep "Image:"
    Image:          nginx:1.23.0

kubectl rollout history deployment nginx --revision=3
deployment.apps/nginx with revision #3
Pod Template:
  Labels:	app=v1
	pod-template-hash=f48dc88cd
  Containers:
   nginx:
    Image:	nginx:1.23.0
    Port:	<none>
    Host Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

10. (Optional) Discuss: Can you foresee potential issues with a rolling deployment? How do you configure a update process that first kills all existing containers with the current version before it starts containers with the new version?
```
  strategy:
    type: Recreate
```

Cleanup
```
k delete deploy nginx
```
