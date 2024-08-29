# Exercise 7

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/), [Pods](https://kubernetes.io/docs/concepts/workloads/pods/), [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

</p>
</details>

Kubernetes runs an init container before the main container. In this lab, the init container retrieves configuration files from a remote location and makes it available to the application running in the main container. The configuration files are shared through a volume mounted by both containers. The running application consumes the configuration files and can render its values.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Adding an Init Container"](https://learning.oreilly.com/scenarios/adding-an-init/9781098163921/).

1. Create a new Pod in a YAML file named `business-app.yaml`. The Pod should define two containers, one init container and one main application container. Name the init container `configurer` and the main container `web`. The init container uses the image `busybox:1.36.1`, the main container uses the image `bmuschko/nodejs-read-config:1.0.0`. Expose the main 

```shell
kubectl run business-app --image=bmuschko/nodejs-read-config:1.0.0 --port=8080 --dry-run=client -o yaml > business-app.yaml
vi business-app.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: business-app
  name: business-app
spec:
  containers:
  - image: bmuschko/nodejs-read-config:1.0.0
    name: web
    ports:
    - containerPort: 8080
    volumeMounts:
    - mountPath: /usr/shared/app
      name: app-volume
  initContainers:
  - image: busybox:1.36.1
    name: configurer
    command:
    - sh
    - -c
    - wget -O /usr/shared/app/config.json https://raw.githubusercontent.com/bmuschko/ckad-crash-course/master/exercises/07-init-container/app/config/config.json
    volumeMounts:
    - mountPath: /usr/shared/app
      name: app-volume
  volumes:
  - name: app-volume
    emptyDir: {}
```

1. container on port 8080.
2. Edit the YAML file by adding a new volume of type `emptyDir` that is mounted at `/usr/shared/app` for both containers.
3. Edit the YAML file by providing the command for the init container. The init container should run a `wget` command for downloading the file `https://raw.githubusercontent.com/bmuschko/ckad-crash-course/master/exercises/07-init-container/app/config/config.json` into the directory `/usr/shared/app`.
4. Start the Pod and ensure that it is up and running.
```shell
k create -f business-app.yaml
k get po business-app -w
```

5. Run the command `curl localhost:8080` from the main application container. The response should render a database URL derived off the information in the configuration file.
```shell
k exec -it business-app web -- /bin/sh
Defaulted container "web" out of: web, configurer (init)
# curl localhost:8080
Database URL: localhost:5432/customers
```

6. (Optional) Discuss: How would you approach debugging a failing command inside of the init container?
```shell
k logs business-app configurer
wget: note: TLS certificate validation not implemented
saving to '/usr/shared/app/config.json'
config.json          100% |********************************|   102  0:00:00 ETA
'/usr/shared/app/config.json' saved
```