# Exercise 17

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

</p>
</details>

In this exercise, you will create a Pod running a NodeJS application. The Pod will define readiness and liveness probes with different parameters.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Creating a Pod with a Readiness Probe of Type HTTP GET Request"](https://learning.oreilly.com/scenarios/creating-a-pod/9781098164102/).

1. Create a new Pod named `hello` with the image `bmuschko/nodejs-hello-world:1.0.0` that exposes the port 3000. Provide the name `nodejs-port` for the container port.
```
k run hello --image=bmuschko/nodejs-hello-world:1.0.0 --port=3000 --dry-run=client -o yaml > pod.yaml
```
2. Add a Readiness Probe that checks the URL path / on the port referenced with the name `nodejs-port` after a 2 seconds delay. You do not have to define the period interval.

[Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
```
```
3. Add a Liveness Probe that verifies that the app is up and running every 8 seconds by checking the URL path / on the port referenced with the name `nodejs-port`. The probe should start with a 5 seconds delay.
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: hello
  name: hello
spec:
  containers:
  - image: bmuschko/nodejs-hello-world:1.0.0
    name: hello
    ports:
    - name: nodejs-port
      containerPort: 3000
    readinessProbe:
      httpGet:
        port: nodejs-port
        path: /
      initialDelaySeconds: 2 
    livenessProbe:
      httpGet:
        port: nodejs-port
        path: /
      initialDelaySeconds: 5 
      periodSeconds: 8
```
```
k apply -f pod.yaml
```
4. Shell into container and curl `localhost:3000`. Write down the output. Exit the container.
```
k exec -it hello -- /bin/sh
# curl localhost:3000
Hello World
```
5. Retrieve the logs from the container. Write down the output.
```
k logs hello    
Magic happens on port 3000
```

Cleanup
```
k delete pod hello
```
