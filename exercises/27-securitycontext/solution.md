# Exercise 27

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

</p>
</details>

In this exercise, you will create a Pod that defines a security context with different options.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Defining a Security Context for a Container"](https://learning.oreilly.com/scenarios/defining-a-security/9781098164263/).

1. Define a Pod named `busybox-security-context` that uses the image `busybox:1.28` for a single container running the command `sh -c sleep 1h`.
```
k run busybox-security-context --image=busybox:1.28 --dry-run=client -o yaml > pod.yaml -- sh -c 'sleep 1h'
```

2. Add an ephemeral Volume of type `emptyDir`. Mount the Volume to the container at `/data/test`.
3. Define a security context that runs the container with user ID 1000, with group ID 3000, and the file system group ID 2000. Ensure that the container should not allow privilege escalation.

[Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-security-context
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - image: busybox:1.28
    name: busybox-security-context
    args:
    - sh
    - -c
    - sleep 1h
    volumeMounts:
    - mountPath: /data/test
      name: data-volume
    securityContext:
      allowPrivilegeEscalation: false
  volumes:
  - name: data-volume
    emptyDir: {}
```

4. Create the Pod object and ensure that it transitions into the "Running" status.
```
k apply -f pod.yaml

k get pod
NAME                       READY   STATUS    RESTARTS   AGE
busybox-security-context   1/1     Running   0          23s
```
5. Open a shell to the running container and create a new file named `logs.txt` in the directory `/data/test`. What's the file's user ID and group ID?
```
k exec -it busybox-security-context -- /bin/sh
/ $ cd /data/test
/data/test $ touch logs.txt
/data/test $ ls -l
total 0
-rw-r--r--    1 1000     2000             0 Sep 10 15:52 logs.txt
```

Cleanup
```
k delete -f pod.yaml
```
