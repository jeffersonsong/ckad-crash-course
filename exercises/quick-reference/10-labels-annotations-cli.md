query for all production Pods that belong to the teams `shiny` and `legacy`
```
k get pods -l 'team in (shiny, legacy)'
NAME       READY   STATUS    RESTARTS   AGE
backend    1/1     Running   0          11m
frontend   1/1     Running   0          12m
```

Remove the label `env` from the `backend` Pod
```
k label pod database env-
```

Render the surrounding 3 lines of YAML of all Pods that have annotations.
```
k get pod -o yaml | grep -C 3 "annotations"
- apiVersion: v1
  kind: Pod
  metadata:
    annotations:
      contact: Mary Harris
    creationTimestamp: "2024-09-07T13:40:26Z"
    labels:
```

Deleye multiple pods at once
```
k delete pods frontend backend database
```
