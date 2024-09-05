# Exercise 29

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `y72`<br>
* Documentation: [Debug Services](https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/)

</p>
</details>

Kate is an developer in charge of implementing a web-based application stack. She is not very familiar with Kubernetes yet, and asked if you could help out. The relevant objects have been created, however, connection to the application cannot be established from within the cluster. Help Kate with fixing the configuration of her YAML manifests.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Troubleshooting a Service"](https://learning.oreilly.com/scenarios/troubleshooting-a-service/9781098164300/).

1. Create the objects from the YAML manifest [setup.yaml](./setup.yaml).

```
k apply -f setup.yaml
namespace/y72 created
deployment.apps/web-app created
service/web-app created
```
2. Inspect the objects in the namespace `y72`.
```
k get pod,deploy,svc -n y72 -o wide --show-labels
NAME                           READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES   LABELS
pod/web-app-744bfb8b47-2jxqr   1/1     Running   0          3m25s   10.244.2.0   minikube   <none>           <none>            app=web-app,pod-template-hash=744bfb8b47
pod/web-app-744bfb8b47-69b2b   1/1     Running   0          3m25s   10.244.2.1   minikube   <none>           <none>            app=web-app,pod-template-hash=744bfb8b47

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES                              SELECTOR      LABELS
deployment.apps/web-app   2/2     2            2           3m25s   web-app      bmuschko/nodejs-hello-world:1.0.0   app=web-app   <none>

NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE     SELECTOR    LABELS
service/web-app   ClusterIP   10.99.93.135   <none>        80/TCP    3m25s   run=myapp   <none>
```

```
k get endpoints -n y72
NAME      ENDPOINTS   AGE
web-app   <none>      71s

```
3. Create a temporary Pod using the image `busybox:1.36.1` in the namespace `y72`. The container command should make a `wget` call to the Service `web-app`. The `wget` will not be able to establish a successful connection to the Service.

```
k run temp --image=busybox:1.36.1 -n y72 -it --rm --restart=Never -- /bin/sh

/ # wget http://10.99.93.135:80
Connecting to 10.99.93.135:80 (10.99.93.135:80)
wget: can't connect to remote host (10.99.93.135): Connection refused
```

```
k logs web-app-744bfb8b47-2jxqr -n y72
Magic happens on port 3000

k logs web-app-744bfb8b47-69b2b -n y72
Magic happens on port 3000
```
4. Identify the root cause for the connection issue and fix it. Verify the correct behavior by repeating the previous step. The `wget` call should return a sucessful response.

```
 k expose deploy web-app -n y72 --port=80 --target-port=3000 --type=ClusterIP --dry-run=client -o yaml > service.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  name: web-app
  namespace: y72
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 3000
  selector:
    app: web-app
  type: ClusterIP
status:
  loadBalancer: {}
```

```
k replace -f service.yaml 
service/web-app replaced

k get endpoints -n y72
NAME      ENDPOINTS                         AGE
web-app   10.244.2.3:3000,10.244.2.4:3000   5m23s
```

```
/ # wget http://10.99.93.135:80
Connecting to 10.99.93.135:80 (10.99.93.135:80)
saving to 'index.html'
index.html           100% |**********************************************************************************************************************************************|    12  0:00:00 ETA
'index.html' saved
```

Cleanup
```
k delete -f setup.yaml 
namespace "y72" deleted
deployment.apps "web-app" deleted
service "web-app" deleted
```