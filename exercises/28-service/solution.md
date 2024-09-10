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
2. Create a Deployment named `myapp` that creates 1 replica running the image `nginx:1.23.4-alpine`. Expose the container port 80.
```
k create deployment myapp --image=nginx:1.23.4-alpine --replicas=1 --port=80

k expose deploy myapp --port=80 --target-port=80 --type=ClusterIP
```
3. Scale the Deployment to 2 replicas.
```
k scale deploy myapp --replicas=2

k get deploy myapp
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
myapp   2/2     2            2           2m15s
```
4. Create a temporary Pod using the image `busybox` and run a `wget` command against the IP of the service.
```
k run temp --image=busybox -it --rm --restart=Never -- wget -O- http://myapp.default.svc
k run temp --image=busybox -it --rm --restart=Never -- wget -O- http://myapp.default.svc.cluster.local

k get svc myapp
NAME    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
myapp   ClusterIP   10.101.235.193   <none>        80/TCP    103s

k run temp --image=busybox -it --rm --restart=Never -- wget -O- http://10.101.235.193
```
5. Change the service type so that the Pods can be reached from outside of the cluster.
```
k expose deploy myapp --port=80 --target-port=80 --type=NodePort

k get svc myapp
NAME    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
myapp   NodePort   10.100.40.84   <none>        80:31726/TCP   17s
```
6. Run a `wget` command against the service from outside of the cluster.
```
minikube ip
192.168.49.2

k get node -o wide
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
minikube   Ready    control-plane   31d   v1.30.0   192.168.49.2   <none>        Ubuntu 22.04.4 LTS   6.8.0-40-generic   docker://26.1.1

wget -O- http://192.168.49.2:31726
```
7. (Optional) Discuss: Can you expose the Pods as a service without a deployment?
Yes
8. (Optional) Discuss: Under what condition would you use the service type `LoadBalancer`?
With cloud provider
