# Exercise 13

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/), [Pods](https://kubernetes.io/docs/concepts/workloads/pods/), [Services](https://kubernetes.io/docs/concepts/services-networking/service/)

</p>
</details>

In this exercise, you will set up a blue-green Deployment scenario. You'll first create the initial (blue) Deployment and expose it will a Service. Later, you will create a second (green) Deployment and switch over traffic.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Implementing the Blue-Green Deployment Strategy"](https://learning.oreilly.com/scenarios/implementing-the-blue-green/9781098164041/).

1. Create a Deployment named `nginx-blue` with 3 replicas. The Pod template of the Deployment should use container image `nginx:1.23.0` and assign the label `version=blue`.

```
k create deployment nginx-blue --replicas=3 --image=nginx:1.23.0 --port=80 --dry-run=client -o yaml > nginx-blue-deployment.yam
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      version: blue
  template:
    metadata:
      labels:
        version: blue
    spec:
      containers:
      - image: nginx:1.23.0
        name: nginx
        ports:
        - containerPort: 80
```
```
k apply -f nginx-blue-deployment.yaml
```

2. Expose the Deployment with a Service of type `ClusterIP` named `nginx`. Map the incoming and outgoing port to 80. Select the Pod with label `version=blue`.

```
kubectl expose deployment nginx-blue --port=80 --target-port=80 --name=nginx --type=ClusterIP
```

3. Run a temporary Pod with the container image `alpine/curl:8.5.0` to make a call against the Service using `curl`.

```
k get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   24d
nginx        ClusterIP   10.104.44.215   <none>        80/TCP    59s

k run curl --rm -it --image=alpine/curl:8.5.0 --restart=Never -- http://10.104.44.215:80 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
pod "curl" deleted
```

4. Create a second Deployment named `nginx-green` with 3 replicas. The Pod template of the Deployment should use container image `nginx:1.23.4` and assign the label `version=green`.
```
cp nginx-blue-deployment.yaml nginx-green-deployment.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-green
spec:
  replicas: 3
  selector:
    matchLabels:
      version: green
  template:
    metadata:
      labels:
        version: green
    spec:
      containers:
      - image: nginx:1.23.0
        name: nginx
        ports:
        - containerPort: 80
```
```
k apply -f nginx-green-deployment.yaml
```

5. Change the Service's label selection so that traffic will be routed to the Pods controlled by the Deployment `nginx-green`.

```
k get svc nginx -o wide
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE     SELECTOR
nginx        ClusterIP   10.104.44.215   <none>        80/TCP    9m46s   version=blue
```

```yaml
  selector:
    version: blue

  selector:
    version: blue
```

```
k get svc nginx -o wide
NAME    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE   SELECTOR
nginx   ClusterIP   10.104.44.215   <none>        80/TCP    10m   version=green
```
6. Delete the Deployment named `nginx-blue`.
```
k delete deploy nginx-blue
```

7. Run a temporary Pod with the container image `alpine/curl:8.5.0` to make a call against the Service.
```
k run curl --rm -it --image=alpine/curl:8.5.0 --restart=Never -- http://10.104.44.215:80 
```
