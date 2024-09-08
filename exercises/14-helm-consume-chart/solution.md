# Exercise 14

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Helm](https://helm.sh/)

</p>
</details>

In this exercise, you use Helm to install Kubernetes objects needed for the open source monitoring solution [Prometheus](https://prometheus.io/). The easiest way to install Prometheus on top of Kubernetes is with the help of the [prometheus-operator](https://prometheus-operator.dev/) Helm chart.

> [!IMPORTANT]
> You will need to have Helm installed on your machine. The Helm documentation page provides detailed, OS-specific [installation instructions](https://helm.sh/docs/intro/install/).

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Installing an Existing Helm Chart from the Central Chart Repository"](https://learning.oreilly.com/scenarios/installing-an-existing/9781098164065/).

1. The Prometheus Helm charts reside in the [artifact repository](https://prometheus-community.github.io/helm-charts). Add the repository to the list of known repositories accessible by Helm with the name `prometheus-community`.
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
"prometheus-community" has been added to your repositories

helm repo list                     
NAME                	URL                                               
prometheus-community	https://prometheus-community.github.io/helm-charts
```
2. Update to the latest information about charts from the respective chart repository.
```
helm repo update prometheus-community
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "prometheus-community" chart repository
Update Complete. ⎈Happy Helming!⎈
```
3. Run the Helm command for listing available Helm charts and their versions. Identify the latest chart version for `kube-prometheus-stack`.
```
helm search repo kube-prometheus-stack
NAME                                      	CHART VERSION	APP VERSION	DESCRIPTION                                       
prometheus-community/kube-prometheus-stack	62.5.1       	v0.76.1    	kube-prometheus-stack collects Kubernetes manif...
```
4. Install the the chart `kube-prometheus-stack`.
```
helm install prometheus prometheus-community/kube-prometheus-stack
NAME: prometheus
LAST DEPLOYED: Sat Sep  7 22:03:39 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace default get pods -l "release=prometheus"

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```
5. List the installed Helm chart.
```
helm list
NAME      	NAMESPACE	REVISION	UPDATED                              	STATUS  	CHART                       	APP VERSION
prometheus	default  	1       	2024-09-07 22:03:39.7838317 -0400 EDT	deployed	kube-prometheus-stack-62.5.1	v0.76.1  
```
6. List the Service named `prometheus-operated` created by the Helm chart. The object resides in the `default` namespace.
```
k get svc prometheus-operated
NAME                  TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
prometheus-operated   ClusterIP   None         <none>        9090/TCP   77s
```
7. Use the kubectl `port-forward` command to forward the local port 8080 to the port 9090 of the Service.
```
k port-forward service/prometheus-operated 8080:9090
Forwarding from 127.0.0.1:8080 -> 9090
Forwarding from [::1]:8080 -> 9090
```
8. Open a browser and bring up the Prometheus dashboard.

9. Stop port forwarding and uninstall the Helm chart.
```
helm uninstall prometheus
release "prometheus" uninstalled
```
