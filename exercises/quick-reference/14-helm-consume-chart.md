```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo list
NAME                  URL
prometheus-community  https://prometheus-community.github.io/helm-charts

helm repo update prometheus-community
helm search repo kube-prometheus-stack
NAME                                        CHART VERSION APP VERSION DESCRIPTION
prometheus-community/kube-prometheus-stack  62.5.1        v0.76.1     kube-prometheus-stack collects Kubernetes manif...

helm install prometheus prometheus-community/kube-prometheus-stack

helm list
NAME        NAMESPACE REVISION  UPDATED                               STATUS    CHART                         APP VERSION
prometheus  default   1         2024-09-07 22:03:39.7838317 -0400 EDT deployed  kube-prometheus-stack-62.5.1  v0.76.1

k port-forward service/prometheus-operated 8080:9090

helm uninstall prometheus
```
