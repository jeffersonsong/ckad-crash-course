# Exercise 22

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Resource Management for Pods and Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/), [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

</p>
</details>

You have been tasked with creating a Pod for running an application in container. During application development, you ran a load test for figuring out the minimum amount of resources needed and the maximum amount of resources the application is allowed to grow to. Define those resource requests and limits for the Pod.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Defining Container Resource Requests and Limits"](https://learning.oreilly.com/scenarios/defining-container-resource/9781098164188/).

1. Define a Pod named `hello-world` running the container image `bmuschko/nodejs-hello-world:1.0.0`. The container exposes the port 3000.

```
k run hello-world --image=bmuschko/nodejs-hello-world:1.0.0 --port=3000 --dry-run=client -o yaml > pod.yaml
```

2. Add a Volume of type `emptyDir` and mount it the container path `/var/log`.
3. For the container, specify the following minimum number of resources as follows:

    - CPU: 100m
    - Memory: 500Mi
    - Ephemeral storage: 1Gi

4. For the container, specify the following maximum number of resources as follows:

    - Memory: 500Mi
    - Ephemeral storage: 2Gi

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: hello-world
  name: hello-world
spec:
  containers:
  - image: bmuschko/nodejs-hello-world:1.0.0
    name: hello-world
    ports:
    - containerPort: 3000
    volumeMounts:
     - mountPath: /var/log
       name: log-volume
    resources: 
      requests:
        cpu: 100m
        memory: 500Mi
        ephemeral-storage: 1Gi
      limits:
        memory: 500Mi
        ephemeral-storage: 2Gi
  volumes:
  - name: log-volume
    emptyDir: {}
```

5. Create the Pod from the YAML manifest.

```
k apply -f pod.yaml
```
6. Inspect the Pod details. Which node does the Pod run on?
```
k describe pod hello-world

Containers:
  hello-world:
    Limits:
      ephemeral-storage:  2Gi
      memory:             500Mi
    Requests:
      cpu:                100m
      ephemeral-storage:  1Gi
      memory:             500Mi
    Mounts:
      /var/log from log-volume (rw)
Volumes:
  log-volume:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
```

Cleanup
```
k delete pod hello-world
```