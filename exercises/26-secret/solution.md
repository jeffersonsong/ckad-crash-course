# Exercise 26

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

</p>
</details>

In this exercise, you will first create a Secret from literal values. Next, you'll create a Pod and consume the Secret as environment variables. Finally, you'll print out its values from within the container.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Creating a Secret and Consuming It as Environment Variables"](https://learning.oreilly.com/scenarios/creating-a-secret/9781098164232/).

1. Create a new Secret named `db-credentials` with the key/value pair `db-password=passwd`.
```
kubectl create secret generic db-credentials --from-literal=db-password=passwd
```
2. Create a Pod named `backend` that uses the Secret as environment variable named `DB_PASSWORD` and runs the container with the image `nginx:1.23.4-alpine`.
```
k run backend --image=nginx:1.23.4-alpine --dry-run=client -o yaml > pod.yaml
```
[Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend
spec:
  containers:
  - image: nginx:1.23.4-alpine
    name: backend
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: db-password
```
```
k apply -f pod.yaml
```

3. Shell into the Pod and print out the created environment variables. You should be able to find the `DB_PASSWORD` variable.
```
k exec -it backend -- /bin/sh          
/ # env | grep DB_PASSWORD
DB_PASSWORD=passwd
```
4. (Optional) Discuss: What is one of the benefit of using a Secret over a ConfigMap?
A Secret is distributed only to the nodes running Pods that actually require access to it. Moreover, Secrets are stored in memory and are never written to a physical storage.

Cleanup
```
k delete -f pod.yaml
```
