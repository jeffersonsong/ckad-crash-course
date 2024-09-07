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
```
k create ns ckad-prep
namespace/ckad-prep created
```
2. In the namespace `ckad-prep`, create a new Pod named `mypod` with the image `nginx:2.3.5`. Expose the port 80.
```
k run mypod -n ckad-prep --image=nginx:2.3.5 --port=80
pod/mypod created
```
3. Identify the issue with creating the container. Write down the root cause of the issue in a file named `pod-error.txt`.
```
k describe pod mypod -n ckad-prep
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  66s                default-scheduler  Successfully assigned ckad-prep/mypod to minikube
  Normal   Pulling    28s (x3 over 66s)  kubelet            Pulling image "nginx:2.3.5"
  Warning  Failed     28s (x3 over 66s)  kubelet            Failed to pull image "nginx:2.3.5": Error response from daemon: Get "https://registry-1.docker.io/v2/": dial tcp: lookup registry-1.docker.io on 192.168.49.1:53: server misbehaving
  Warning  Failed     28s (x3 over 66s)  kubelet            Error: ErrImagePull

```
4. Change the image of the Pod to `nginx:1.15.12`.
```
k set image pod/mypod -n ckad-prep mypod=nginx:1.15.12 
pod/mypod image updated
```
5. List the Pod and ensure that the container is running.
```
k get pod -n ckad-prep
NAME    READY   STATUS    RESTARTS   AGE
mypod   1/1     Running   0          4m32s
```
6. Log into the container and run the `ls` command. Write down the output. Log out of the container.
```
k exec -it mypod -n ckad-prep -- /bin/sh
# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```
7. Retrieve the IP address of the Pod `mypod`.
```
k get pod mypod -n ckad-prep -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP          NODE       NOMINATED NODE   READINESS GATES
mypod   1/1     Running   0          6m42s   10.0.0.98   minikube   <none>           <none>
```
8. Run a temporary Pod in the namespace `ckad-prep` using the image `busybox`, shell into it and run a `wget` command against the `nginx` Pod using port 80.
```
k run temp -it --image=busybox --rm --restart=Never -- wget http://10.0.0.98
Connecting to 10.0.0.98 (10.0.0.98:80)
saving to 'index.html'
index.html           100% |********************************|   612  0:00:00 ETA
'index.html' saved
pod "temp" deleted
```
9. Render the logs of Pod `mypod`.
```
k logs mypod -n ckad-prep            
10.0.0.239 - - [07/Sep/2024:01:25:27 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
```
10. Delete the Pod and the namespace.
```
k delete pod mypod -n ckad-prep
k delete ns ckad-prep
```
