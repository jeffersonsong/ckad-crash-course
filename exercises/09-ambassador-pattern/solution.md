# Exercise 9

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `ext-access`<br>
* Documentation: [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)

</p>
</details>

In this example, you'll be asked to implement rate-limiting functionality for HTTP(S) calls to an external service. For example, the requirements for the rate limiter could say that an application can only make a maximum of five calls every 15 minutes. Instead of strongly coupling the rate-limiting logic to the application code, it will be provided by an ambassador container.

The image `bmuschko/nodejs-business-app:1.0.0` represents a Node.js-based application that makes a call to localhost on port 8081. The ambassador container represented by the image `bmuschko/nodejs-ambassador:1.0.0` running on port 8081 will take on making the HTTP call to the external service while at the same time enforcing rate limiting.

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Implementing the Ambassador Pattern"](https://learning.oreilly.com/scenarios/implementing-the-ambassador/9781098163945/).

1. Create a YAML manifest for a Pod with the name `rate-limiter` in the namespace `ext-access`. Store the definition in the file named `rate-limiter.yaml`. The main application container named `business-app` should use the image `bmuschko/nodejs-business-app:1.0.0` and expose the container port 8080.

k create ns ext-access
namespace/ext-access created

2. Modify the YAML manifest by adding the ambassador container named `ambassador`. The ambassador container uses the image `bmuschko/nodejs-ambassador:1.0.0` and exposes the container port 8081.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: rate-limiter
  namespace: ext-access
spec:
  containers:
  - image: bmuschko/nodejs-business-app:1.0.0
    name: business-app
    ports:
    - containerPort: 8080
  - image: bmuschko/nodejs-ambassador:1.0.0
    name: ambassador
    ports:
    - containerPort: 8081
  restartPolicy: Never
```

```shell
k apply -f rate-limiter.yaml 
pod/rate-limiter created

k get po -n ext-access -w          
NAME           READY   STATUS    RESTARTS   AGE
rate-limiter   2/2     Running   0          88s
```

3. Test the rate-limiting functionality. First, create the Pod, shell into the `business-app` container that runs the business application, and execute a series of `curl` commands that target `localhost:8081/test`, the end point of the `ambassador` container. The first five calls will be allowed to the external service. On the sixth call, you’ll receive an error message, as the rate limit has been reached within the given time frame.

```shell
k exec -it rate-limiter -n ext-access -c business-app -- /bin/sh

# curl http://localhost:8081/test
```
```json
{
  "args": {
    "test": "undefined"
  },
  "headers": {
    "host": "postman-echo.com",
    "x-forwarded-proto": "http",
    "x-request-start": "t=1725242560.048",
    "connection": "close",
    "x-forwarded-port": "443",
    "x-amzn-trace-id": "Root=1-66d51cc0-3ad4c08e7cd9e55d72d1cc1a"
  },
  "url": "http://postman-echo.com/get?test=undefined"
}
```

```shell
# curl http://localhost:8081/test
# curl http://localhost:8081/test
# curl http://localhost:8081/test
# curl http://localhost:8081/test

# curl http://localhost:8081/test
Too many requests made created from this IP, please try again after an hour
```
