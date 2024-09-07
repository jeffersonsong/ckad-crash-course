# Exercise 3

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/)

</p>
</details>

In this exercise, you will create a parallel-executed Job that is in charge of writing a randomly-generated string in base64-encoded form to standard output.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive labs ["Creating a Nonparallel Job"](https://learning.oreilly.com/scenarios/creating-a-nonparallel/9781098163877/), and ["Creating a Parallel Job"](https://learning.oreilly.com/scenarios/creating-a-parallel/9781098163884/).

1. Create a Job named `random-hash` that executes the shell command `echo $RANDOM | base64 | head -c 20`. Configure the Job to execute with two Pods in parallel. The number of completions should be set to five.
```
kubectl create job random-hash --image=alpine --dry-run=client -o yaml -- /bin/sh -c 'echo $RANDOM | base64 | head -c 20' > job.yaml
```
[Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/)
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: random-hash
spec:
  completions: 5
  parallelism: 2
  template:
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - "echo $RANDOM | base64 | head -c 20"
        image: alpine
        name: random-hash
      restartPolicy: Never
```
```
k apply -f job.yaml 
job.batch/random-hash created

k get job random-hash -w
NAME          STATUS     COMPLETIONS   DURATION   AGE
random-hash   Complete   5/5           13s        14s
```
2. Identify the Pods that executed the shell command. How many Pods do expect to exist?
```
k get pod
NAME                READY   STATUS      RESTARTS   AGE
random-hash-9fs8b   0/1     Completed   0          18s
random-hash-bfrx4   0/1     Completed   0          17s
random-hash-ggkcd   0/1     Completed   0          23s
random-hash-qhg9d   0/1     Completed   0          23s
random-hash-rv45b   0/1     Completed   0          14s
```
3. Retrieve the generated hash from one of the Pods.
```
k logs random-hash-9fs8b
NjQxMgo=
```

4. Delete the Job. Will the corresponding Pods continue to exist?
```
k delete job random-hash
job.batch "random-hash" deleted

k get pod 
No resources found in default namespace.
```
