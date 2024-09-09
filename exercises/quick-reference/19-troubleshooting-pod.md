[Debugging with an ephemeral debug container](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/#ephemeral-container)
```
kubectl debug -it date-recorder --image=busybox --target=debian --share-processes
```
