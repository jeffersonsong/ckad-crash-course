Service DNS
```
myapp.default.svc
myapp.default.svc.cluster.local
```

Access NodePort
```
k get node -o wide
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
minikube   Ready    control-plane   31d   v1.30.0   192.168.49.2   <none>        Ubuntu 22.04.4 LTS   6.8.0-40-generic   docker://26.1.1

wget -O- http://192.168.49.2:31726
```
