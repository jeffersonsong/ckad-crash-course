# Exercise 2

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `ckad-prep`<br>
* Documentation: [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)

</p>
</details>

In this exercise, you will practice the creation of a new Pod in a namespace. Once created, you will inspect it, shell into it and run some operations inside of the container.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive labs ["Creating and Interacting with a Pod in a Namespace"](https://learning.oreilly.com/scenarios/creating-and-interacting/9781098163846/), ["Creating a Pod that Uses a Custom Command"](https://learning.oreilly.com/scenarios/creating-a-pod/9781098163853/), and ["Modifying the Configuration of an Existing Pod"](https://learning.oreilly.com/scenarios/modifying-the-configuration/9781098163860/).

1. Create the namespace `ckad-prep`.
```shell
k create ns ckad-prep 
```

2. In the namespace `ckad-prep`, create a new Pod named `mypod` with the image `nginx:2.3.5`. Expose the port 80.
```shell
k run mypod -n ckad-prep --port=80 --image=nginx:2.3.5
```

3. Identify the issue with creating the container. Write down the root cause of the issue in a file named `pod-error.txt`.
4. Change the image of the Pod to `nginx:1.15.12`.
```shell
k edit po mypod -n ckad-prep
```

5. List the Pod and ensure that the container is running.
```shell
k get po -n ckad-prep
```

6. Log into the container and run the `ls` command. Write down the output. Log out of the container.
```shell
k exec -it mypod -n ckad-prep -- /bin/sh
# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

7. Retrieve the IP address of the Pod `mypod`.
```shell
k get po -n ckad-prep -o wide       
NAME    READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
mypod   1/1     Running   0          3m26s   10.244.1.56   minikube   <none>           <none>
```

8. Run a temporary Pod in the namespace `ckad-prep` using the image `busybox`, shell into it and run a `wget` command against the `nginx` Pod using port 80.
```shell
kubectl run -it busybox -n ckad-prep --image=busybox --restart=Never --rm -- /bin/sh
If you don't see a command prompt, try pressing enter.
/ # 
/ # 
/ # wget http://10.244.1.56
```

9. Render the logs of Pod `mypod`.
```shell
k logs mypod -n ckad-prep
10.244.1.54 - - [27/Aug/2024:02:09:13 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
```

10. Delete the Pod and the namespace.
```shell
k delete po mypod -n ckad-prep
k delete ns ckad-prep
```
