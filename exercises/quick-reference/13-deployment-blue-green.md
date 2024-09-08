Run a temporary Pod with the container image `alpine/curl:8.5.0` to make a call against the Service.
```
k run curl --image=alpine/curl:8.5.0 -it --rm --restart=Never -- curl 10.99.80.18
```
