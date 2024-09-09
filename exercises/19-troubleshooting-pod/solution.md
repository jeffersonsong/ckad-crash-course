# Exercise 19

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Debug Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/)

</p>
</details>

In this exercise, you will practice your troubleshooting skills by inspecting a misconfigured Pod.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Troubleshooting a Pod"](https://learning.oreilly.com/scenarios/troubleshooting-a-pod/9781098164140/).

1. Create a new Pod from the YAML manifest in the file [`pod.yaml`](./pod.yaml).
```
k apply -f pod.yaml 
pod/date-recorder created
```
2. Check the Pod's status. Do you see any issue?
```
k get pod date-recorder
NAME            READY   STATUS    RESTARTS   AGE
date-recorder   1/1     Running   0          55s

k describe pod date-recorder
    State:          Running
      Started:      Sun, 08 Sep 2024 22:23:21 -0400
    Ready:          True

Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
```
3. Render the logs of the running container and identify an issue.
```
k logs date-recorder 
[Error: ENOENT: no such file or directory, open '/root/tmp/startup-marker.txt'] {
  errno: -2,
  code: 'ENOENT',
  syscall: 'open',
  path: '/root/tmp/startup-marker.txt'
}
```
4. Shell into the container. Can you verify the issue based on the rendered log message?
[Debugging with an ephemeral debug container](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/#ephemeral-container)
```
k exec -it date-recorder -- /bin/sh
OCI runtime exec failed: exec failed: unable to start container process: exec: "/bin/sh": stat /bin/sh: no such file or directory: unknown
command terminated with exit code 126

kubectl debug -it date-recorder --image=busybox --target=debian --share-processes
~ # ps aux
PID   USER     TIME  COMMAND
    1 root      6:42 /nodejs/bin/node -e const fs = require('fs'); let timestamp = Date.now(); fs.writeFile('/root/tmp/startup-marker.txt', timestamp.toString(), err => { if (err) { console.error(err); } while(true) {} });
   23 root      0:00 /bin/sh
   31 root      0:00 ps aux

~ # ls /root/tmp
ls: /root/tmp: No such file or directory
```
5. Suggest solutions that can fix the root cause of the issue.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: date-recorder
spec:
  containers:
  - name: debian
    image: gcr.io/distroless/nodejs20-debian11
    command: ["/nodejs/bin/node", "-e", "const fs = require('fs'); let timestamp = Date.now(); fs.writeFile('/root/tmp/startup-marker.txt', timestamp.toString(), err => { if (err) { console.error(err); } while(true) {} });"]
    volumeMounts:
    - mountPath: /root/tmp
      name: storage
  volumes:
  - name: storage
    emptyDir: {}
```

Cleanup
```
k delete -f pod.yaml
```
