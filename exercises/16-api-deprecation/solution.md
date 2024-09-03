# Exercise 16

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Kubernetes Deprecation Policy](https://kubernetes.io/docs/reference/using-api/deprecation-policy/), [Deprecated API Migration Guide](https://kubernetes.io/docs/reference/using-api/deprecation-guide/)

</p>
</details>

The Kubernetes administrator in charge of the cluster is planning to upgrade all nodes from Kubernetes 1.8 to 1.26. You, as the application developer, implemented Kubernetes YAML manifests that operate an application stack. The administrator provided you with a Kubernetes 1.26 test environment. Make sure that the YAML manifests are compatible with Kubernetes version 1.26.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Identifying and Replacing a Deprecated API"](https://learning.oreilly.com/scenarios/identifying-and-replacing/9781098164096/).

1. Inspect the files [`deployment.yaml`](./deployment.yaml) and [`configmap.yaml`](./configmap.yaml) in the current directory.
2. Create the objects from the YAML manifests. Make modifications to the definitions as needed.

```
k apply -f configmap.yaml 
configmap/data-config created

k apply -f deployment.yaml                                                                                      
error: resource mapping not found for name: "nginx" namespace: "" from "deployment.yaml": no matches for kind "Deployment" in version "apps/v1beta2"
ensure CRDs are installed first

k api-resources | grep deploy
deployments                         deploy       apps/v1                           true         Deployment

k apply -f deployment.yaml           
The Deployment "nginx" is invalid: 
* spec.selector: Required value
* spec.template.metadata.labels: Invalid value: map[string]string{"run":"app"}: `selector` does not match template `labels`

k apply -f deployment.yaml
deployment.apps/nginx created
```

3. Verify that the objects could be instantiated with Kubernetes 1.26.

```
k get cm,deploy
NAME                         DATA   AGE
configmap/data-config        2      9m15s
configmap/kube-root-ca.crt   1      24d

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   2/2     2            2           3m1s
```
