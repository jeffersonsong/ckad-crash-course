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
2. Build the container image from the `Dockerfile` with the tag `nodejs-hello-world:1.0.0`.
```shell
docker build -t nodejs-hello-world:1.0.0 .
```

3. Run a container with the container image. Make the application available on port 80.
```shell
docker run -p 80:3000 nodejs-hello-world:1.0.0
```

4. Execute a `curl` or `wget` command against the application's endpoint.
```shell
curl "http://localhost:80"
```

5. Retrieve the container logs.
```shell
docker ps
CONTAINER ID   IMAGE                      COMMAND                  CREATED              STATUS              PORTS                                   NAMES
935ddf290ea6   nodejs-hello-world:1.0.0   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:80->3000/tcp, :::80->3000/tcp   intelligent_curie

docker logs 935ddf290ea6
```

6. Save the container image to the file `nodejs-hello-world-1.0.0.tar`.
```shell
docker image ls
REPOSITORY                    TAG       IMAGE ID       CREATED          SIZE
nodejs-hello-world            1.0.0     f60b01fe9cf7   18 minutes ago   180MB
```
```shell
docker image save --output nodejs-hello-world-1.0.0.tar f60b01fe9cf7
docker image save -o nodejs-hello-world-1.0.0.tar nodejs-hello-world:1.0.0
```
```shell
docker stop 935ddf290ea6
```