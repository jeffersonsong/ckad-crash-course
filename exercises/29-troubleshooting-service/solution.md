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
```
2. Inspect the objects in the namespace `y72`.
```
k get all -n y72 -o wide --show-labels
NAME                           READY   STATUS    RESTARTS   AGE   IP          NODE       NOMINATED NODE   READINESS GATES   LABELS
pod/web-app-744bfb8b47-282gm   1/1     Running   0          12m   10.0.0.78   minikube   <none>           <none>            app=web-app,pod-template-hash=744bfb8b47
pod/web-app-744bfb8b47-6h7dd   1/1     Running   0          12m   10.0.0.36   minikube   <none>           <none>            app=web-app,pod-template-hash=744bfb8b47

NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE   SELECTOR    LABELS
service/web-app   ClusterIP   10.100.84.63   <none>        80/TCP    12m   run=myapp   <none>

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                              SELECTOR      LABELS
deployment.apps/web-app   2/2     2            2           12m   web-app      bmuschko/nodejs-hello-world:1.0.0   app=web-app   <none>

NAME                                 DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                              SELECTOR                                   LABELS
replicaset.apps/web-app-744bfb8b47   2         2         2       12m   web-app      bmuschko/nodejs-hello-world:1.0.0   app=web-app,pod-template-hash=744bfb8b47   app=web-app,pod-template-hash=744bfb8b47
```
3. Create a temporary Pod using the image `busybox:1.36.1` in the namespace `y72`. The container command should make a `wget` call to the Service `web-app`. The `wget` will not be able to establish a successful connection to the Service.
```
k run temp -n y72 --image=busybox:1.36.1 -it --rm --restart=Never -- /bin/sh
If you don't see a command prompt, try pressing enter.
/ # wget -O- http://10.100.84.63
Connecting to 10.100.84.63 (10.100.84.63:80)
wget: can't connect to remote host (10.100.84.63): Connection refused
```
4. Identify the root cause for the connection issue and fix it. Verify the correct behavior by repeating the previous step. The `wget` call should return a sucessful response.
```yaml
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: web-app
```
```
k run temp -n y72 --image=busybox:1.36.1 -it --rm --restart=Never -- /bin/sh
If you don't see a command prompt, try pressing enter.
/ # wget -O- http://10.100.84.63
Connecting to 10.100.84.63 (10.100.84.63:80)
writing to stdout
Hello World
-                    100% |**********************************************************************************************************************************************************************************************|    12  0:00:00 ETA
written to stdout
```

Cleanup
```
k delete -f setup.yaml
```
