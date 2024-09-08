[Update CHANGE-CAUSE rollout description for a specific revision](https://discuss.kubernetes.io/t/update-change-cause-rollout-description-for-a-specific-revision/18722)

Update the image to `nginx:1.23.4`.
```
k set image deploy/nginx nginx=nginx:1.23.4
```
Assign the change cause "Pick up patch version" to the revision.

```
k annotate deploy/nginx kubernetes.io/change-cause='Pick up patch version'

k rollout history deploy/nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
2         Pick up patch version
```

Revert the Deployment to revision 1
```
kubectl rollout undo deploy/nginx --to-revision=1

k rollout history deploy/nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
2         Pick up patch version
3         <none>
```
