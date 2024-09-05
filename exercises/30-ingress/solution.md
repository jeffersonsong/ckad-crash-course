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
k create deploy web --image=bmuschko/nodejs-hello-world:1.0.0 --replicas=1 --port=3000
```
3. Expose the Deployment with a Service named `web` of type `ClusterIP`. The Service routes traffic to the Pods controlled by the Deployment `web`.

```
k expose deployment web --type=ClusterIP
```

4. Make a request to the endpoint of the application on the context path `/`. You should see the message "Hello World".
```
k get svc web
NAME   TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
web    ClusterIP   10.96.76.4   <none>        3000/TCP   45s

k get endpoints web
NAME   ENDPOINTS         AGE
web    10.244.2.6:3000   51s

k run temp --image=alpine/curl:8.5.0 -it --rm --restart=Never -- /bin/sh
/ # curl http://10.96.76.4:3000
Hello World
```

5. Create an Ingress that exposes the path `/` for the host `hello-world.exposed`. The traffic should be routed to the Service created earlier.

```
kubectl create ingress ingress --rule="hello-world.exposed/*=web:3000"
```
6. List the Ingress object.

```
k describe ingress ingress
Name:             ingress
Labels:           <none>
Namespace:        default
Address:          
Ingress Class:    nginx
Default backend:  <default>
Rules:
  Host                 Path  Backends
  ----                 ----  --------
  hello-world.exposed  
                       /   web:3000 (10.244.2.6:3000)
Annotations:           <none>
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  Sync    22s   nginx-ingress-controller  Scheduled for sync
```
7. Add an entry in `/etc/hosts` that maps the virtual node IP address to the host `hello-world.exposed`.
```
minikube ip
192.168.49.2

vi /etc/hosts
192.168.49.2    hello-world.exposed
```

8. Make a request to `http://hello-world.exposed`. You should see the message "Hello World".

```
curl http://hello-world.exposed/
Hello World
```

Cleanup
````
k delete ingress ingress
k delete svc web
k delete deploy web
```