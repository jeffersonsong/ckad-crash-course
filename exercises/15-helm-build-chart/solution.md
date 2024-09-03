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

```
helm create hello-world
```

1. Create a new chart file named `Chart.yaml`. Define all mandatory attributes including the chart's API version, the name, and the version. Add the following key-value pairs: `API version: 1.0.0`, `Name: web-app`, and `Version: 2.5.4`.

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
k run hello-world --image=muschko/nodejs-hello-world:1.0.0 --port=3000 --dry-run=client -o yaml > web-app-pod.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: hello-world
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
cp web-app-pod.yaml web-app-pod-teplate.yaml
```

```yaml
    - containerPort: {{ .Values.container_port }}
```

4. Create the template file `web-app-service-template.yaml`. The YAML manifest defines a Service named `web-app-service` of type `ClusterIP`. The target port should use the placeholder `container_port` from the `values.yaml` file, the port should use the placeholder `service_port` from the `values.yaml` file.

```
kubectl expose pod hello-world --port=80 --target-port=3000 --name=web-app-service --type=ClusterIP --dry-run=client -o yaml > web-app-service.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    name: hello-world
  name: web-app-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 3000
  selector:
    name: hello-world
  type: ClusterIP
```
```
k apply -f web-app-service.yaml
cp web-app-service.yaml web-app-service-template.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    name: hello-world
  name: web-app-service
spec:
  ports:
  - port: {{ .Values.service_port }}
    protocol: TCP
    targetPort: {{ .Values.container_port }}
  selector:
    name: hello-world
  type: ClusterIP
```

```
k run curl --rm -it --image=alpine/curl:8.5.0 --restart=Never -- http://10.104.44.215:80 
k delete svc web-app-service
k delete pod hello-world
rm web-app-pod.yaml web-app-service.yaml
```

5. Bundle the template files into a chart archive file by running the correct Helm command. What's the name of the file produced?

```
helm lint .
==> Linting .
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed


helm template .
---
# Source: web-app/templates/web-app-service-template.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    name: hello-world
  name: web-app-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 3000
  selector:
    name: hello-world
  type: ClusterIP
---
# Source: web-app/templates/web-app-pod-template.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: hello-world
  name: hello-world
spec:
  containers:
  - image: bmuschko/nodejs-hello-world:1.0.0
    name: hello-world
    ports:
    - containerPort: 3000
```

```
helm package .
Successfully packaged chart and saved it to: /home/jeffersons/Tutorials/udemy.com/ckad-crash-course/exercises/15-helm-build-chart/hello-world/web-app-2.5.4.tgz
```

6. Install the chart with the name `hello-world` to the namespace `app-stack` using the appropriate Helm command. Override the value of `service_port` with the value `9090`.
```
helm install --namespace app-stack --create-namespace --set service_port=9090 web-app ./web-app-2.5.4.tgz
NAME: web-app
LAST DEPLOYED: Mon Sep  2 23:06:26 2024
NAMESPACE: app-stack
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

7. Find the installed objects provided by the chart.

```
helm list -n app-stack
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
hello-world     app-stack       1               2024-09-02 23:18:56.64931289 -0400 EDT  deployed        web-app-2.5.4   1.0.0  

k get all -n app-stack
NAME              READY   STATUS    RESTARTS   AGE
pod/hello-world   1/1     Running   0          31s

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/web-app-service   ClusterIP   10.106.115.136   <none>        9090/TCP   31s
```

```
k run curl --rm -it --image=alpine/curl --restart=Never -- http://10.106.115.136:9090
Hello World
pod "curl" deleted
```

8. Delete the Helm chart.
```
helm uninstall hello-world -n app-stack
release "hello-world" uninstalled

```