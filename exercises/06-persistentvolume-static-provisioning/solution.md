# Exercise 6

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

</p>
</details>

In this exercise, you will create a PersistentVolume, connect it to a PersistentVolumeClaim and mount the claim to a specific path of a Pod.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Creating and Using a PersistentVolume with Static Provisioning"](https://learning.oreilly.com/scenarios/creating-and-using/9781098163914/).

1. Create a PersistentVolume named `pv`, access mode `ReadWriteMany`, 512Mi of storage capacity and the host path `/data/config`.

```
vi pv.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  capacity:
    storage: 512Mi
  accessModes:
  - ReadWriteMany
  hostPath:
    path: /data/config
    type: DirectoryOrCreate
```

```
k create -f pv.yaml
k get pv
```

2. Create a PersistentVolumeClaim named `pvc`. The claim should request 256Mi and use an empty string value for the storage class. Ensure that the PersistentVolumeClaim is properly bound after its creation.

```
vi pvc.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
spec:
  storageClassName: ""
  volumeName: pv
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 256Mi
```

```
k create -f pvc.yaml
k get pvc
```

3. Mount the PersistentVolumeClaim from a new Pod named `app` with the path `/var/app/config`. The Pod uses the image `nginx:1.21.6`.
```
k run app --image=nginx:1.21.6 --dry-run=client -o yaml > pod.yaml
vi pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - image: nginx:1.21.6
    name: app
    volumeMounts:
    - mountPath: /var/app/config
      name: pv-volume
  volumes:
  - name: pv-volume
    persistentVolumeClaim:
      claimName: pvc
```

```
k create -f pod.yaml
```

4. Open an interactive shell to the Pod and create a file in the directory `/var/app/config`.
```
k get po
k exec -it app -- /bin/sh
# echo "Hello world!" > /var/app/config/hello.txt
```