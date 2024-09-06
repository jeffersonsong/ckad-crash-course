# Exercise 4

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [CronJobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)

</p>
</details>

In this exercise, you will create a CronJob and render its executions.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Creating and Inspecting a Periodic Operation Using a CronJob"](https://learning.oreilly.com/scenarios/creating-and-inspecting/9781098163891/).

1. Create a CronJob named `current-date` that runs every minute and executes the shell command `echo "Current date: $(date)"`.
```
k create cronjob current-date --image=alpine --schedule="* * * * *" -- echo "Current date: $(date)"
k create cronjob current-date --image=alpine --schedule="* * * * *" -- /bin/sh -c 'echo "Current date: $(date)"'
```

2. Watch the jobs as they are being scheduled.
```
k get job -w
```

3. Identify one of the Pods that ran the CronJob and render the logs.
```
k get pod
k get po --show-labels
k logs pod/current-date-28746846-8z6gd
```

4. Determine the number of successful executions the CronJob will keep in its history.
```
k get cronjob current-date -o yaml | grep successfulJobsHistoryLimit
```

5. Delete the CronJob.
```
k delete cronjob current-date
```
