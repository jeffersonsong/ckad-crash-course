```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  storageClassName: ""
  capacity:
    storage: 512Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /data/config
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 256Mi
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: app
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
k exec -it app -- /bin/sh
# touch /var/app/config/test.txt
```
```
minikube ssh
docker@minikube:~$ ls /data/config
test.txt
```

