# Application Design and Build


## Define, build and modify container images

dockerfile

```FROM openjdk:11-jre-slim
WORKDIR /app
COPY target/java-hello-world-0.0.1.jar java-hello-world.jar
ENTRYPOINT ["java", "-jar", "/app/java-hello-world.jar"]
EXPOSE 8080
```

```
$ docker build -t java-hello-world:1.0.0 .
```

```
$ docker images
```

This tells docker to forward port 8081 on local host to port 8080 on the docker image:

```
$ docker run -d -p 8081:8080 java-hello-world:1.0.0
```

```
$ curl localhost:8081
```

```
$ docker login --username=eric
```

Tag and push image to docker:

```
$ docker tag java-hello-world:1.0.0 eric/java-hello-world:1.0.0
$ docker push eric/java-hello-world:1.0.0
```


## Understand Jobs and CronJobs

## Understand multi-container Pod design patterns (e.g. sidecar, init and others)

## Utilize persistent and ephemeral volumes