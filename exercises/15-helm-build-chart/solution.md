# Exercise 15

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Helm](https://helm.sh/)

</p>
</details>

In this exercise, you will practice the implementation, packaging, and installation of a configurable custom Helm chart.

> [!IMPORTANT]
> You will need to have Helm installed on your machine. The Helm documentation page provides detailed, OS-specific [installation instructions](https://helm.sh/docs/intro/install/).

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Implementing, Packaging, and Installing a Custom Helm Chart"](https://learning.oreilly.com/scenarios/implementing-packaging-and/9781098164072/).

1. Create a new chart file named `Chart.yaml`. Define all mandatory attributes including the chart's API version, the name, and the version. Add the following key-value pairs: `API version: 1.0.0`, `Name: web-app`, and `Version: 2.5.4`.
[Using Helm](https://helm.sh/docs/intro/using_helm/)

```
helm create web-app
```
```yaml
apiVersion: v2
name: web-app
type: application
version: 2.5.4
appVersion: "1.0.0"
```
2. Create a new values file named `values.yaml`. It should contain the following key-value pairs: `service_port: 80`, and `container_port: 3000`.
```yaml
service_port: 80
container_port: 3000
```
3. Create the template file `web-app-pod-template.yaml`. The YAML manifest defines a Pod named `hello-world` with the image `bmuschko/nodejs-hello-world:1.0.0`. The container port uses the placeholder `container_port` from the `values.yaml` file.
```
k run hello-world --image=bmuschko/nodejs-hello-world:1.0.0 --port=3000 --dry-run=client -o yaml > web-app-pod.yaml
```
web-app-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: hello-world
  name: hello-world
spec:
  containers:
  - image: bmuschko/nodejs-hello-world:1.0.0
    name: hello-world
    ports:
    - containerPort: 3000
```
```
k apply -f web-app-pod.yaml
```
4. Create the template file `web-app-service-template.yaml`. The YAML manifest defines a Service named `web-app-service` of type `ClusterIP`. The target port should use the placeholder `container_port` from the `values.yaml` file, the port should use the placeholder `service_port` from the `values.yaml` file.
```
k expose pod hello-world --port=80 --target-port=3000 --type=CluserIP --name=web-app-service --dry-run=client -o yaml > web-app-service.yaml
```
web-app-service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello-world
  name: web-app-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 3000
  selector:
    app: hello-world
  type: ClusterIP
```
```
k apply -f web-app-service.yaml
k get svc web-app-service
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
web-app-service   ClusterIP   10.100.147.109   <none>        80/TCP    35s

k port-forward svc/web-app-service 8080:80
Forwarding from 127.0.0.1:8080 -> 3000
Forwarding from [::1]:8080 -> 3000

curl localhost:8080
Hello World

k delete svc web-app-service
k delete pod hello-world

mv web-app-pod.yaml web-app-pod-template.yaml
mv web-app-service.yaml web-app-service-template.yaml
```
web-app-pod-template.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: hello-world
  name: hello-world
spec:
  containers:
  - image: bmuschko/nodejs-hello-world:1.0.0
    name: hello-world
    ports:
    - containerPort: {{ .Values.container_port }}
```
web-app-service-template.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello-world
  name: web-app-service
spec:
  ports:
  - port: {{ .Values.service_port }}
    protocol: TCP
    targetPort: {{ .Values.container_port }}
  selector:
    app: hello-world
  type: ClusterIP
```
```
tree
.
├── charts
├── Chart.yaml
├── templates
│   ├── web-app-pod-template.yaml
│   └── web-app-service-template.yaml
└── values.yaml
```

5. Bundle the template files into a chart archive file by running the correct Helm command. What's the name of the file produced?
```
helm package web-app
Successfully packaged chart and saved it to: /home/jeffersons/Tutorials/udemy.com/ckad-crash-course/exercises/15-helm-build-chart/web-app-2.5.4.tgz
```
6. Install the chart with the name `hello-world` to the namespace `app-stack` using the appropriate Helm command. Override the value of `service_port` with the value `9090`.
```
helm install --set service_port=9090 -n app-stack --create-namespace hello-world ./web-app-2.5.4.tgz
NAME: hello-world
LAST DEPLOYED: Sat Sep  7 22:53:20 2024
NAMESPACE: app-stack
STATUS: deployed
REVISION: 1
TEST SUITE: None
```
7. Find the installed objects provided by the chart.
```
k get all -n app-stack
NAME              READY   STATUS    RESTARTS   AGE
pod/hello-world   1/1     Running   0          25s

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/web-app-service   ClusterIP   10.102.131.178   <none>        9090/TCP   25s

k port-forward -n app-stack svc/web-app-service 8080:9090
Forwarding from 127.0.0.1:8080 -> 3000
Forwarding from [::1]:8080 -> 3000

curl localhost:8080
Hello World
```
8. Delete the Helm chart.
```
helm list -n app-stack
NAME       	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
hello-world	app-stack	1       	2024-09-07 22:53:20.410723681 -0400 EDT	deployed	web-app-2.5.4	1.0.0 

helm uninstall hello-world -n app-stack
release "hello-world" uninstalled

k delete ns app-stack
namespace "app-stack" deleted
```
