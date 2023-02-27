# Application Design and Build


## Define, build and modify container images

dockerfile

```FROM openjdk:11-jre-slim
WORKDIR /app
COPY target/java-hello-world-0.0.1.jar java-hello-world.jar
ENTRYPOINT ["java", "-jar", "/app/java-hello-world.jar"]
EXPOSE 8080
```


## Understand Jobs and CronJobs

## Understand multi-container Pod design patterns (e.g. sidecar, init and others)

## Utilize persistent and ephemeral volumes