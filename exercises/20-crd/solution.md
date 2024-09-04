# Exercise 20

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Custom Resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)

</p>
</details>

As an application developer, you may want to install Kubernetes functionality that exends the platform using the Kubernetes operator pattern. The objective of this exercise is to familiarize yourself with creating and managing CRDs. You will not need to write a controller.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Defining and Interacting with a CRD"](https://learning.oreilly.com/scenarios/defining-and-interacting/9781098164164/).

1. Create a CRD resource named `backup.example.com` with the following specification:

    - Group: `example.com`
    - Version: `v1`
    - Kind: `Backup`
    - Singular: `backup`
    - Plural: `backups`
    - Properties of type `string`: `cronExpression`, `podName`, `path`

[Extend the Kubernetes API with CustomResourceDefinitions](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: backups.example.com
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              cronExpression: 
                type: string
              podName: 
                type: string
              path: 
                type: string
  scope: Namespaced
  names:
    plural: backups
    singular: backup
    kind: Backup
```
```
k apply -f backup-resourcedefinition.yaml
```

2. Retrieve the details for the `Backup` custom resource created in the previous step.

```
k get crd backups.example.com     
NAME                  CREATED AT
backups.example.com   2024-09-04T01:37:31Z

k describe crd backups.example.com

k api-resources | grep backups
backups                                          example.com/v1                    true         Backup
```

3. Create a custom object named `nginx-backup` for the CRD. Provide the following property values:

    - `cronExpression`: `0 0 * * *`
    - `podName`: `nginx`
    - `path`: `/usr/local/nginx`

```yaml
apiVersion: example.com/v1
kind: Backup
metadata:
  name: nginx-backup
spec:
  cronExpression: '0 0 * * *'
  podName: nginx
  path: /usr/local/nginx
```

```
k apply -f nginx-backup.yaml 
backup.example.com/nginx-backup created
```

4. Retrieve the details for the `nginx-backup` object created in the previous step.

```
k get backup
NAME           AGE
nginx-backup   18s

k describe backup nginx-backup
Name:         nginx-backup
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  example.com/v1
Kind:         Backup
Metadata:
  Creation Timestamp:  2024-09-04T01:41:50Z
  Generation:          1
  Resource Version:    729886
  UID:                 79574568-7fae-463f-9ef0-6d9617a86d77
Spec:
  Cron Expression:  0 0 * * *
  Path:             /usr/local/nginx
  Pod Name:         nginx
Events:             <none>
```