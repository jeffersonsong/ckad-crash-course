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
kubectl create job random-hash --image=busybox --dry-run=client -o yaml > random-hash-job.yaml
```

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
      - image: alpine
        name: random-hash
        command: ["/bin/sh", "-c", "echo $RANDOM | base64 | head -c 20"]
      restartPolicy: Never
```

```
k create -f random-hash-job.yaml
```

2. Identify the Pods that executed the shell command. How many Pods do expect to exist?
```
k get po
```

3. Retrieve the generated hash from one of the Pods.
```
k logs pod/random-hash-974sq
MTkxNzIK
```

4. Delete the Job. Will the corresponding Pods continue to exist?
```
k delete job random-hash
k get po
```
