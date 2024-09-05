# Exercise 28

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Services](https://kubernetes.io/docs/concepts/services-networking/service/)

</p>
</details>

In this exercise, you will create a Deployment and expose a container port for its Pods. You will demonstrate the differences between the service types ClusterIP and NodePort.

> [!IMPORTANT]
> If you are using minikube, the network is limited if using the Docker driver on Darwin, Windows, or WSL, and the Node IP is not reachable directly. Refer to the [documentation](https://minikube.sigs.k8s.io/docs/handbook/accessing/#nodeport-access) to fetch the minikube IP and a service's node port.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive labs ["Creating a Service of Type ClusterIP"](https://learning.oreilly.com/scenarios/creating-a-service/9781098164287/) and ["Creating a Service of Type NodePort"](https://learning.oreilly.com/scenarios/creating-a-service/9781098164294/).

1. Create a Service named `myapp` of type `ClusterIP` that exposes port 80 and maps to the target port 80.
```
kubectl expose deployment myapp --port=80 --target-port=80 --type=ClusterIP --name=myapp
service/myapp exposed

k get svc myapp -o wide
NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE   SELECTOR
myapp   ClusterIP   10.105.88.77   <none>        80/TCP    36s   app=myapp
```

2. Create a Deployment named `myapp` that creates 1 replica running the image `nginx:1.23.4-alpine`. Expose the container port 80.

```
kubectl create deployment myapp --image=nginx:1.23.4-alpine --port=80                                           
deployment.apps/myapp created
```

3. Scale the Deployment to 2 replicas.
```
k scale deploy/myapp --replicas=2
deployment.apps/myapp scaled
```

4. Create a temporary Pod using the image `busybox` and run a `wget` command against the IP of the service.

```
k run tmp -it --rm --image=busybox -- /bin/sh
/ # wget http://10.105.88.77:80
Connecting to 10.105.88.77:80 (10.105.88.77:80)
saving to 'index.html'
index.html           100% |**********************************************************************************************************************************************************************************************|   615  0:00:00 ETA
'index.html' saved
/ # rm index.html
/ # wget http://myapp.default.svc.cluster.local:80
Connecting to myapp.default.svc.cluster.local:80 (10.105.88.77:80)
saving to 'index.html'
index.html           100% |**********************************************************************************************************************************************************************************************|   615  0:00:00 ETA
'index.html' saved
```

5. Change the service type so that the Pods can be reached from outside of the cluster.
```
k delete svc myapp
kubectl expose deployment myapp --port=80 --target-port=80 --type=NodePort --name=myapp
service/myapp exposed

k get svc myapp
NAME    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
myapp   NodePort   10.107.107.47   <none>        80:32197/TCP   9s
```

6. Run a `wget` command against the service from outside of the cluster.

```
minikube service myapp --url
http://192.168.49.2:32197

curl http://192.168.49.2:32197
```

7. (Optional) Discuss: Can you expose the Pods as a service without a deployment?

Yes

8. (Optional) Discuss: Under what condition would you use the service type `LoadBalancer`?

Provision a load balancer for your service in the cloud.

Cleanup
```
k delete svc myapp
k delete deploy myapp
```