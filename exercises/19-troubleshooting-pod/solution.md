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
k get po date-recorder        
NAME            READY   STATUS    RESTARTS   AGE
date-recorder   1/1     Running   0          105s

k describe po date-recorder
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

```
k exec -it pod/date-recorder -- /bin/sh
OCI runtime exec failed: exec failed: unable to start container process: exec: "/bin/sh": stat /bin/sh: no such file or directory: unknown
command terminated with exit code 126

k debug date-recorder -it --image=busybox --target=debian --share-processes=true
Targeting container "debian". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
Defaulting debug container name to debugger-8mfqm.
If you don't see a command prompt, try pressing enter.
/ # 
/ # 
/ # ps aux
PID   USER     TIME  COMMAND
    1 root      9:52 /nodejs/bin/node -e const fs = require('fs'); let timestamp = Date.now(); fs.writeFile('/root/tmp/startup-marker.txt', timestamp.toString(), err => { if (err) { console.error(err); } while(true) {} });
   47 root      0:00 sh
   53 root      0:00 ps aux
/ # ls /root/tmp
ls: /root/tmp: No such file or directory
```

5. Suggest solutions that can fix the root cause of the issue.

```
k delete pod date-recorder --grace-period=0 --force
```

Mount ephemeral directory /root/tmp for output.
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
      name: output-volume
  volumes:
  - name: output-volume
    emptyDir: {}
```