# Exercise 30

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Ingresses](https://kubernetes.io/docs/concepts/services-networking/ingress/), [Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)

</p>
</details>

In this exercise, you will create an Ingress with a simple rule that routes traffic to a Service.

> [!IMPORTANT]
> Kubernetes requires running an Ingress Controller to evaluate Ingress rules. Make sure your cluster employs an [Ingress Controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/). You can find installation guidance in the file [ingress-controller-setup.md](./ingress-controller-setup.md). If you are using minikube, the network is limited if using the Docker driver on Darwin, Windows, or WSL, and the Node IP is not reachable directly. Refer to the [documentation](https://minikube.sigs.k8s.io/docs/handbook/accessing/) to gain access to the minikube IP.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Defining and Using an Ingress"](https://learning.oreilly.com/scenarios/defining-and-using/9781098164317/).

1. Verify that the Ingress Controller is running.
2. Create a new Deployment named `web` that controls a single replica running the image `bmuschko/nodejs-hello-world:1.0.0` on port 3000.
```
k create deployment web --image=bmuschko/nodejs-hello-world:1.0.0 --port=3000
```
3. Expose the Deployment with a Service named `web` of type `ClusterIP`. The Service routes traffic to the Pods controlled by the Deployment `web`.
```
k expose deployment web --type=ClusterIP
```
4. Make a request to the endpoint of the application on the context path `/`. You should see the message "Hello World".
```
k run temp --image=alpine/curl -it --rm --restart=Never -- curl http://10.110.2.219:3000
Hello World
pod "temp" deleted

k run temp --image=alpine/curl -it --rm --restart=Never -- curl http://web.default.svc:3000
```
5. Create an Ingress that exposes the path `/` for the host `hello-world.exposed`. The traffic should be routed to the Service created earlier.
```
kubectl create ingress hello-world-ingress --rule="hello-world.exposed/=web:80"
```
6. List the Ingress object.
```
k get ingress -o wide
NAME                  CLASS   HOSTS                 ADDRESS        PORTS   AGE
hello-world-ingress   nginx   hello-world.exposed   192.168.49.2   80      48s
```
7. Add an entry in `/etc/hosts` that maps the virtual node IP address to the host `hello-world.exposed`.
```
k get node -o wide
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
minikube   Ready    control-plane   31d   v1.30.0   192.168.49.2   <none>        Ubuntu 22.04.4 LTS   6.8.0-40-generic   docker://26.1.1

sudo vi /etc/hosts
192.168.49.2    hello-world.exposed
```
8. Make a request to `http://hello-world.exposed`. You should see the message "Hello World".

Cleanup
```
k delete -f ingress.yaml
k delete -f service.yaml
k delete -f deployment.yaml
```

