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
kubectl create cronjob current-date --image=alpine --schedule="*/1 * * * *" --dry-run=client -o yaml -- /bin/sh -c `echo "Current date: $(date)"` > cronjob.yaml
```
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: current-date
spec:
  jobTemplate:
    metadata:
      name: current-date
    spec:
      template:
        spec:
          containers:
          - command:
            - /bin/sh
            - -c
            - 'echo "Current date: $(date)"'
            image: alpine
            name: current-date
          restartPolicy: OnFailure
  schedule: '*/1 * * * *'
```
```
k apply -f cronjob.yaml 
cronjob.batch/current-date created
```
2. Watch the jobs as they are being scheduled.
```
k describe cj current-date
Last Schedule Time:  Fri, 06 Sep 2024 22:07:00 -0400
Active Jobs:         <none>
Events:
  Type    Reason            Age    From                Message
  ----    ------            ----   ----                -------
  Normal  SuccessfulCreate  10m    cronjob-controller  Created job current-date-28761237
  Normal  SawCompletedJob   10m    cronjob-controller  Saw completed job: current-date-28761237, status: Complete
  Normal  SuccessfulCreate  9m41s  cronjob-controller  Created job current-date-28761238
  Normal  SawCompletedJob   9m37s  cronjob-controller  Saw completed job: current-date-28761238, status: Complete
```
3. Identify one of the Pods that ran the CronJob and render the logs.
```
k logs current-date-28761246-7d68s
Current date: Sat Sep  7 02:06:01 UTC 2024
```

4. Determine the number of successful executions the CronJob will keep in its history.
```
k describe cj current-date
Successful Job History Limit:  3
```
5. Delete the CronJob.
```
k delete cj current-date
cronjob.batch "current-date" deleted
```
