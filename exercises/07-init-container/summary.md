```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
spec:
  initContainers:
  - image: busybox:1.36.1
    name: configurer
    command:
    - /bin/sh
    - -c
    - "sleep 5 && wget -O /usr/shared/app/config.json https://raw.githubusercontent.com/bmuschko/ckad-crash-course/master/exercises/07-init-container/app/config/config.json"
    volumeMounts:
    - mountPath: /usr/shared/app
      name: storage
  containers:
  - image: bmuschko/nodejs-read-config:1.0.0
    name: web
    ports:
    - containerPort: 8080
    volumeMounts:
    - mountPath: /usr/shared/app
      name: storage
  volumes:
  - name: storage
    emptyDir: {}
```
