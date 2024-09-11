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
```
2. Inspect the objects in the namespace `k1` and `k2`.
3. Determine the virtual IP address of Pod `nginx` in namespace `k2`. Try to make a `wget` call on port 80 from the Pod `busybox` in namespace `k1` to the Pod `nginx` in namespace `k2`. The call will fail with the current setup.
```
k get pod -n k2 -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          96s   10.0.0.193   minikube   <none>           <none>

k exec -it busybox -n k1 -- /bin/sh
/ # 
/ # wget -O- http://10.0.0.193:80
Connecting to 10.0.0.193:80 (10.0.0.193:80)
```
4. Create a network policy that allows performing ingress calls for all Pods in namespace `k1` to the Pod `nginx` in namespace `k2`. Pods in all other namespaces should be denied to make ingress calls to Pods in namespace `k2`.

[Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: network-policy
  namespace: k2
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: k1
    ports:
    - protocol: TCP
      port: 80
```
```
k apply -f network-policy.yaml
```

5. Repeat step 3 to verify that a network connection can be established.
```
k exec -it busybox -n k1 -- /bin/sh
/ # wget -O- http://10.0.0.193:80
Connecting to 10.0.0.193:80 (10.0.0.193:80)
writing to stdout
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
-                    100% |**********************************************************************************************************************************************************************************************|   615  0:00:00 ETA
written to stdout
```
