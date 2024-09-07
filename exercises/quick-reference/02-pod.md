```
k set image pod/mypod -n ckad-prep mypod=nginx:1.15.12
k run temp -it --image=busybox --rm --restart=Never -- /bin/sh
# wget -O- 192.168.60.149:80
```
