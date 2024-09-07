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

1. Create a new Pod in a YAML file named `business-app.yaml`. The Pod should define two containers, one init container and one main application container. Name the init container `configurer` and the main container `web`. The init container uses the image `busybox:1.36.1`, the main container uses the image `bmuschko/nodejs-read-config:1.0.0`. Expose the main container on port 8080.
```
k run web --image=bmuschko/nodejs-read-config:1.0.0 --port=8080 --dry-run=client -o yaml > business-app.yaml
```

2. Edit the YAML file by adding a new volume of type `emptyDir` that is mounted at `/usr/shared/app` for both containers.
3. Edit the YAML file by providing the command for the init container. The init container should run a `wget` command for downloading the file `https://raw.githubusercontent.com/bmuschko/ckad-crash-course/master/exercises/07-init-container/app/config/config.json` into the directory `/usr/shared/app`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: web
  name: web
spec:
  initContainers:
  - image: busybox:1.36.1
    name: configurer
    command:
    - /bin/sh
    - -c
    - "sleep 5 && wget -O /usr/shared/app/config.json https://raw.githubusercontent.com/bmuschko/ckad-crash-course/master/exercises/07-init-container/app/config/config.json"
    volumeMounts:
    - mountPath: /usr/shared/app
      name: storage
  containers:
  - image: bmuschko/nodejs-read-config:1.0.0
    name: web
    ports:
    - containerPort: 8080
    volumeMounts:
    - mountPath: /usr/shared/app
      name: storage
  volumes:
  - name: storage
    emptyDir: {}
```
4. Start the Pod and ensure that it is up and running.
```
k get pod         
NAME   READY   STATUS    RESTARTS   AGE
web    1/1     Running   0          61s
```
5. Run the command `curl localhost:8080` from the main application container. The response should render a database URL derived off the information in the configuration file.
```
k exec -it web web -- /bin/sh
Defaulted container "web" out of: web, configurer (init)
# curl localhost:8080
Database URL: localhost:5432/customers
```

6. (Optional) Discuss: How would you approach debugging a failing command inside of the init container?
```
k describe po web | less

Init Containers:
  configurer:
    Container ID:  docker://83f01ba9544ef1462ff6064685fb6a5ce1b9d6b25b7e21038b51a837ebc9f9fc
    Image:         busybox:1.36.1
    Image ID:      docker-pullable://busybox@sha256:9ae97d36d26566ff84e8893c64a6dc4fe8ca6d1144bf5b87b2b85a32def253c7
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
      -c
      sleep 5 && wget -O /usr/shared/app/config.json https://raw.githubusercontent.com/bmuschko/ckad-crash-course/master/exercises/07-init-container/app/config/config.json
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Fri, 06 Sep 2024 23:44:04 -0400
      Finished:     Fri, 06 Sep 2024 23:44:09 -0400
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/shared/app from storage (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-r4lqx (ro)

Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m50s  default-scheduler  Successfully assigned default/web to minikube
  Normal  Pulled     3m49s  kubelet            Container image "busybox:1.36.1" already present on machine
  Normal  Created    3m49s  kubelet            Created container configurer
  Normal  Started    3m49s  kubelet            Started container configurer
```

Cleanup
```
k delete -f business-app.yaml
```
