# Exercise 25

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/), [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

</p>
</details>

In this exercise, you will first create a ConfigMap from a YAML configuration file as a source. Later, you'll create a Pod, consume the ConfigMap as Volume and inspect the key-value pairs as files.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Creating a ConfigMap and Consuming It as Volume"](https://learning.oreilly.com/scenarios/creating-a-configmap/9781098164225/).

1. Inspect the YAML configuration file named [`application.yaml`](./application.yaml).
2. Create a new ConfigMap named `app-config` from that file.

```
kubectl create configmap app-config --from-file=application.yaml 
configmap/app-config created
```

3. Create a Pod named `backend` that consumes the ConfigMap as Volume at the mount path `/etc/config`. The container runs the image `nginx:1.23.4-alpine`.
[Populate a Volume with data stored in a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#populate-a-volume-with-data-stored-in-a-configmap)

```
k run backend --image=nginx:1.23.4-alpine --dry-run=client -o yaml > pod.yaml

```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: backend
  name: backend
spec:
  containers:
  - image: nginx:1.23.4-alpine
    name: backend
    volumeMounts:
    - mountPath: /etc/config
      name: config-volume
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

```
k apply -f pod.yaml
pod/backend created
```

4. Shell into the Pod and inspect the file at the mounted Volume path.
```
k exec -it backend -- /bin/sh
/ # cat /etc/config/application.yaml 
dev:
  url: http://dev.bar.com
  name: Developer Setup
prod:
  url: http://foo.bar.com
  name: My Cool App
```

5. (Optional) Discuss: How would you approach hot reloading of values defined by a ConfigMap consumed by an application running in Pod?

[Mounted ConfigMaps are updated automatically](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#mounted-configmaps-are-updated-automatically)