[Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

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
spec:
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

