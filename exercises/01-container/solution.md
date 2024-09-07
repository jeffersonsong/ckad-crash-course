# Exercise 1

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: N/A<br>
* Documentation: [Containerize an application](https://docs.docker.com/get-started/02_our_app/)

</p>
</details>

In this exercise, you will practice building a container image from an existing `Dockerfile`. Then you will run the container from the image, and interact with it. You can use a container builder of your choice, e.g. [buildkit](https://github.com/moby/buildkit), [Podman](https://podman.io/).

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Defining, Building, and Running a Container Image"](https://learning.oreilly.com/scenarios/defining-building-and/9781098163839/).

1. Inspect the [`Dockerfile`](./app/Dockerfile) in the `app` directory.
```
FROM node:20.1.0-alpine
WORKDIR /app
COPY ["package.json", "package-lock.json*", "./"]
RUN npm install --production
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```
2. Build the container image from the `Dockerfile` with the tag `nodejs-hello-world:1.0.0`.
```
docker build -t nodejs-hello-world:1.0.0 .
[+] Building 0.4s (10/10) FINISHED                                                                                                                                             docker:default
 => [internal] load build definition from Dockerfile                                                                                                                                     0.0s
 => => transferring dockerfile: 200B                                                                                                                                                     0.0s
 => [internal] load metadata for docker.io/library/node:20.1.0-alpine                                                                                                                    0.3s
 => [internal] load .dockerignore                                                                                                                                                        0.0s
 => => transferring context: 66B                                                                                                                                                         0.0s
 => [1/5] FROM docker.io/library/node:20.1.0-alpine@sha256:6e56967f8a4032f084856bad4185088711d25b2c2c705af84f57a522c84d123b                                                              0.0s
 => [internal] load build context                                                                                                                                                        0.0s
 => => transferring context: 124B                                                                                                                                                        0.0s
 => CACHED [2/5] WORKDIR /app                                                                                                                                                            0.0s
 => CACHED [3/5] COPY [package.json, package-lock.json*, ./]                                                                                                                             0.0s
 => CACHED [4/5] RUN npm install --production                                                                                                                                            0.0s
 => CACHED [5/5] COPY . .                                                                                                                                                                0.0s
 => exporting to image                                                                                                                                                                   0.0s
 => => exporting layers                                                                                                                                                                  0.0s
 => => writing image sha256:f60b01fe9cf71c77f9d8ee9d8949f894ecf32262a9b3ef1c72374d9506922b52                                                                                             0.0s
 => => naming to docker.io/library/nodejs-hello-world:1.0.0                                                                                                                              0.0s
```
3. Run a container with the container image. Make the application available on port 80.
```
docker run -p 80:3000 nodejs-hello-world:1.0.0
Magic happens on port 3000

```
4. Execute a `curl` or `wget` command against the application's endpoint.
```
curl localhost
Hello World

```
5. Retrieve the container logs.
```
docker logs 21750f1bb6b9
Magic happens on port 3000
```
6. Save the container image to the file `nodejs-hello-world-1.0.0.tar`.
```
docker image save nodejs-hello-world:1.0.0 -o nodejs-hello-world-1.0.0.tar
```

Cleanup
```
docker image rm nodejs-hello-world:1.0.0
```
