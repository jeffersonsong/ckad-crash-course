# Exercise 8

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Pods](https://kubernetes.io/docs/concepts/workloads/pods/), [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

</p>
</details>

The adapter pattern helps with providing a simplified, homogenized view of an application running within a container. For example, we could stand up another container that unifies the log output of the application container. As a result, other monitoring tools can rely on a standardized view of the log output without having to transform it into an expected format.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Creating a Sidecar Container"](https://learning.oreilly.com/scenarios/creating-a-sidecar/9781098163938/).

1. Create a new Pod in a YAML file named `adapter.yaml`. The Pod declares two containers. The container `app` uses the image `busybox:1.36.1` and runs the command `while true; do echo "$(date) | $(du -sh ~)" >> /var/logs/diskspace.txt; sleep 5; done;`. The adapter container `transformer` uses the image `busybox:1.36.1` and runs the command `sleep 20; while true; do while read LINE; do echo "$LINE" | cut -f2 -d"|" >> $(date +%Y-%m-%d-%H-%M-%S)-transformed.txt; done < /var/logs/diskspace.txt; sleep 20; done;` to strip the log output off the date for later consumption by a monitoring tool. Be aware that the logic does not handle corner cases (e.g. automatically deleting old entries) and would look different in production systems.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: adapter-pod
spec:
  containers:
  - image: busybox:1.36.1
    name: app
    command:
    - /bin/sh
    - -c
    - 'while true; do echo "$(date) | $(du -sh ~)" >> /var/logs/diskspace.txt; sleep 5; done;'
    volumeMounts:
    - name: log-volume
      mountPath: /var/logs
  - image: busybox:1.36.1
    name: transformer
    command:
    - /bin/sh
    - -c
    - 'sleep 20; while true; do while read LINE; do echo "$LINE" | cut -f2 -d"|" >> $(date +%Y-%m-%d-%H-%M-%S)-transformed.txt; done < /var/logs/diskspace.txt; sleep 20; done;'
    volumeMounts:
    - name: log-volume
      mountPath: /var/logs
  volumes:
  - name: log-volume
    emptyDir: {}
```

```
k apply -f adapter.yaml

k get po
NAME          READY   STATUS    RESTARTS   AGE
adapter-pod   2/2     Running   0          4m32s
```

2. Before creating the Pod, define an `emptyDir` volume. Mount the volume in both containers with the path `/var/logs`.
3. Create the Pod, log into the container `transformer`. The current directory should continuously write a new file every 20 seconds.

```
k exec -it adapter-pod -c transformer -- /bin/sh
/ # ls
2024-09-02-01-46-40-transformed.txt  2024-09-02-01-48-01-transformed.txt  2024-09-02-01-49-02-transformed.txt  home                                 root                                 var
2024-09-02-01-47-00-transformed.txt  2024-09-02-01-48-21-transformed.txt  bin                                  lib                                  sys
2024-09-02-01-47-21-transformed.txt  2024-09-02-01-48-41-transformed.txt  dev                                  lib64                                tmp
2024-09-02-01-47-41-transformed.txt  2024-09-02-01-48-42-transformed.txt  etc                                  proc                                 usr

/ # cat 2024-09-02-01-46-40-transformed.txt
 4.0K	/root
 4.0K	/root
 4.0K	/root
 4.0K	/root
 4.0K	/root

/ # tail -f /var/logs/diskspace.txt 
Mon Sep  2 01:49:16 UTC 2024 | 4.0K	/root
Mon Sep  2 01:49:21 UTC 2024 | 4.0K	/root
Mon Sep  2 01:49:26 UTC 2024 | 4.0K	/root
Mon Sep  2 01:49:31 UTC 2024 | 4.0K	/root
Mon Sep  2 01:49:36 UTC 2024 | 4.0K	/root
Mon Sep  2 01:49:41 UTC 2024 | 4.0K	/root
Mon Sep  2 01:49:46 UTC 2024 | 4.0K	/root
Mon Sep  2 01:49:51 UTC 2024 | 4.0K	/root
Mon Sep  2 01:49:56 UTC 2024 | 4.0K	/root
Mon Sep  2 01:50:01 UTC 2024 | 4.0K	/root
Mon Sep  2 01:50:06 UTC 2024 | 4.0K	/root
```
