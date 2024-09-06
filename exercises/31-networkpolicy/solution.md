# Exercise 31

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `k1`, `k2`<br>
* Documentation: [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/), [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)

</p>
</details>

All ingress Pod-to-Pod communication has been denied across all namespaces. You want to allow the Pod `busybox` in namespace `k1` to communicate with Pod `nginx` in namespace `k2`. You'll create a network policy to achieve that.

> [!IMPORTANT]
> Without a network policy controller, network policies won't have any effect. You need to configure a network overlay solution that provides this controller. You'll have to go through some extra steps to install and enable the network provider Cilium. Without adhering to the proper prerequisites, network policies won't have any effect. You can find installation guidance in the file [cilium-setup.md](./cilium-setup.md).

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Restricting Access to and from a Pod with Network Policies"](https://learning.oreilly.com/scenarios/restricting-access-to/9781098164324/).

1. Create the objects from the YAML manifest [setup.yaml](./setup.yaml).
```
k apply -f setup.yaml
namespace/k1 created
namespace/k2 created
pod/busybox created
pod/nginx created
networkpolicy.networking.k8s.io/default-deny-ingress created
```

2. Inspect the objects in the namespace `k1` and `k2`.
```
k get pod -n k1 -o wide --show-labels
NAME      READY   STATUS    RESTARTS   AGE   IP         NODE       NOMINATED NODE   READINESS GATES   LABELS
busybox   1/1     Running   0          32m   10.0.0.1   minikube   <none>           <none>            app=frontend

k get pod -n k2 -o wide --show-labels
NAME    READY   STATUS    RESTARTS   AGE   IP          NODE       NOMINATED NODE   READINESS GATES   LABELS
nginx   1/1     Running   0          31m   10.0.0.70   minikube   <none>           <none>            app=backend


k get netpol -n k2 -o wide
NAME                   POD-SELECTOR   AGE
default-deny-ingress   <none>         119s
```

3. Determine the virtual IP address of Pod `nginx` in namespace `k2`. Try to make a `wget` call on port 80 from the Pod `busybox` in namespace `k1` to the Pod `nginx` in namespace `k2`. The call will fail with the current setup.

```
k exec -it busybox -n k1 -- /bin/sh
/ # 
/ # wget --timeout=5 http://10.0.0.70:80
Connecting to 10.0.0.70:80 (10.0.0.70:80)
wget: can't connect to remote host (10.0.0.70): Connection timed out
```

4. Create a network policy that allows performing ingress calls for all Pods in namespace `k1` to the Pod `nginx` in namespace `k2`. Pods in all other namespaces should be denied to make ingress calls to Pods in namespace `k2`.

[Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-ingress-from-k1-pods
  namespace: k2
spec:
  podSelector: 
    matchLabels:
      app: backend
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: k1
    ports:
    - port: 80
      protocol: TCP
  policyTypes:
  - Ingress
```

```
k apply -f network-policy.yaml
networkpolicy.networking.k8s.io/allow-ingress-from-k1-pods created
```

5. Repeat step 3 to verify that a network connection can be established.
```
k exec -it busybox -n k1 -- /bin/sh
/ # 
/ # wget --timeout=5 http://10.0.0.70:80
Connecting to 10.0.0.70:80 (10.0.0.70:80)
saving to 'index.html'
index.html           100% |**********************************************************************************************************************************************************************************************|   615  0:00:00 ETA
'index.html' saved
```

Cleanup

```
k delete -f network-policy.yaml
k delete -f setup.yaml
```