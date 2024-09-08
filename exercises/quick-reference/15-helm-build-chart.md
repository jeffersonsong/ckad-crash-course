```
helm create web-app
```
Chart.yaml
```yaml
apiVersion: v2
name: web-app
type: application
version: 2.5.4
appVersion: "1.0.0"
```
values.yaml
```yaml
service_port: 80
container_port: 3000
```
```
cd templates
k run hello-world --image=bmuschko/nodejs-hello-world:1.0.0 --port=3000 --dry-run=client -o yaml > web-app-pod.yaml
k expose pod hello-world --port=80 --target-port=3000 --type=CluserIP --name=web-app-service --dry-run=client -o yaml > web-app-service.yaml

mv web-app-pod.yaml web-app-pod-template.yaml
mv web-app-service.yaml web-app-service-template.yaml
```
templates/web-app-pod-template.yaml
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
templates/web-app-service-template.yaml
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

```
helm package web-app
helm install --set service_port=9090 -n app-stack --create-namespace hello-world ./web-app-2.5.4.tgz
```
