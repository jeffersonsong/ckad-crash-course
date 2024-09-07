```
docker build -t nodejs-hello-world:1.0.0 .
docker run -p 80:3000 nodejs-hello-world:1.0.0
docker logs 21750f1bb6b9
docker image save nodejs-hello-world:1.0.0 -o nodejs-hello-world-1.0.0.tar
docker image rm nodejs-hello-world:1.0.0
```
Dockerfile
```
FROM node:20.1.0-alpine
WORKDIR /app
COPY ["package.json", "package-lock.json*", "./"]
RUN npm install --production
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

