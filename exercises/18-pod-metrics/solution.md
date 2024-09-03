# Exercise 18

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `stress-test`<br>
* Documentation: [Metrics Server](https://github.com/kubernetes-sigs/metrics-server)

</p>
</details>

> [!IMPORTANT]
> You will need to install the metrics server if you want to be able to inspect actual resource metrics. You can find [installation instructions](https://github.com/kubernetes-sigs/metrics-server#installation) on the project's GitHub page.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Determining Metrics for Pods"](https://learning.oreilly.com/scenarios/determining-metrics-for/9781098164133/).

1. Create the namespace `stress-test`.

```
k create ns stress-test
namespace/stress-test created
```

2. The current directory contains the YAML manifests for three Pods, [`stress-1-pod.yaml`](./stress-1-pod.yaml), [`stress-2-pod.yaml`](./stress-2-pod.yaml), and [`stress-3-pod.yaml`](./stress-3-pod.yaml). Create all of them.

```
k apply -f .
pod/stress-1 created
pod/stress-2 created
pod/stress-3 created
```
3. Use the data available through the metrics server to identify which of the Pods consumes the most memory.

```
k top pod -n stress-test
NAME       CPU(cores)   MEMORY(bytes)   
stress-1   42m          58Mi            
stress-2   56m          105Mi           
stress-3   37m          53M
```

```
k delete pod stress-1 -n stress-test --grace-period=0 --force
k delete pod stress-2 -n stress-test --grace-period=0 --force
k delete pod stress-3 -n stress-test --grace-period=0 --force
k delete ns stress-test
```