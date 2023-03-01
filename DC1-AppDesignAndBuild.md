# Application Design and Build


## Define, build and modify container images

dockerfile:

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

### Create Pods in Kubernetes


```
$ kubectl run java-hello-world --image=eric/java-hello-world:1.0.0 --restart=Never \
--port=8080 --env="DNS_DOMAIN=cluster" --labels="app=hello-world,env=prod"
```

Creating pod from YAML:
```
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
  labels: 
    app: hello-world
    env: prod
spec:
  containers:
  - env:
    - name: DNS_DOMAIN
      value: cluster
    image: eric/java-hello-world:1.0.0
    name: hello-world
    ports: 
    - containerPort: 8080
  restartPolicy: Never

```

```
$ kubectl apply -f pod.yaml
```


List pods:
```
$ kubectl get pods
$ kubectl get pods hello-world
```

### Pod Lifecyle

- Pending  (or Unknown).  
- Running
- Succeeded or Failed

Get Pod Details
```
$ k describe pod hello-world
$ k describe pod hello-world | grep Image:
$ k get pod hello-world -o yaml
```

### Pod Logs

```
k logs hello-world
```


### Exec into Container

```
$ k exec -it hello-world -- /bin/sh
$ k exec -it hello-world -- powershell
$ k exec -it hello-world -- pwsh
```

Run a single command non-interactively

```
$ k exec hello-world -- env
```

### Delete a Pod

```
$ k delete pod hello-world
$ k delete -f pod.yaml
```

### Namespaces

```
k get namespaces
k get ns
k create ns hello-world
```

ns.yaml:
```
apiVersion: v1
kind: Namespace
metadata:
  name: code-red
```

## Understand Jobs and CronJobs

Jobs run until completion and then stay around until their spec.ttlSecondsAfterFinished threshold has expired.

Default is to run a single pod once.   That's a "non-parallel" job.

controlled by 
- spec.template.spec.completions
- spec.template.spec.parallelism


```
$ k create job counter --image=nginx -- /bin/sh -c 'count=0; \
while [ $counter -lt 3]; do counter=$((counter+1)); echo "$counter"; \
sleep 3; done;'
```

```
apiVersion: batch/v1
kind: Job
metadata:
  name: counter
spec:
  template:
  spec:
    containers:
    - name: counter
      image: nginx
      command:
      - /bin/sh
      - -c
      - count=0; while [ $counter -lt 3]; do counter=$((counter+1)); \
        echo "$counter"; sleep 3; done;
    restartPolicy: Never
```

### Job Operation Types

| **Type**                               | **spec.completions** | **spec.parallelism** | **Description**                                                                             |
|----------------------------------------|----------------------|------------------|-----------------------------------------------------------------------------------------|
| Non-parallel job with one completion  | 1                    | 1                | Completes as soon as Pod terminates successfully                                              |
| Parallel with a fixed completion count | >= 1                 | >= 1             | Completes when specified number of tasks finish successfully                            |
| Parallel with worker queue             | unset                | >= 1             | Completes when at least one Pod has terminated successfully and all Pods are terminated |

### Restart Behavior

spec.backoffLimit - number of retries.  default is 6

Restart Policy - spec.template.spec.restartPolicy
- Always (default) - tells k8s to restart the job even if it is successful
- Never - creates new pod
- OnFailure - restarts current pod

### CronJob

this runs every hour, on the hour:
```
$ k create cronjob get-the-date --schedule="0 * * * *" --image=nginx \
-- /bin/sh -c 'echo "it is now: $(date)"'
```

```
k get cronjobs
```

minute - hour - dayofmonth - month - dayofweek (0-6, sun-sat)



## Understand multi-container Pod design patterns (e.g. sidecar, init and others)

## Utilize persistent and ephemeral volumes