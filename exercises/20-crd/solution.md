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

[Create a CustomResourceDefinition](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#create-a-customresourcedefinition)

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
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
    shortNames:
    - bu
```

```
k apply -f crd.yaml 
customresourcedefinition.apiextensions.k8s.io/backups.example.com created
```

2. Retrieve the details for the `Backup` custom resource created in the previous step.
```
k get crd backups.example.com -o yaml
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
  cronExpression: 0 0 * * *
  podName: nginx
  path: /usr/local/nginx
```
```
k apply -f backup.yaml      
backup.example.com/nginx-backup created
```

4. Retrieve the details for the `nginx-backup` object created in the previous step.
```
k describe backup nginx-backup
```

Cleanup
```
k delete -f backup.yaml
k delete -f crd.yaml
```
