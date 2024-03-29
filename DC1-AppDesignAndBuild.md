# Application Design and Build


## Define, build and modify container images

dockerfile:

```
FROM openjdk:11-jre-slim
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
local:remote

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


Create a pod yaml manifest from an image without running it:
```
$ kubectl run nginx --image=nginx --restart=Never --dry-run=client -n mynamespace -o yaml > pod.yaml
```
dry-run=client just previews a command without actually running it


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

### Labels, Selectors and Annotations

#### Labels


Difference between Labels and Annontation
- Labels can be used for querying and selectors.  restricted format
- Annotations are descriptive metadata, but can't be used for querying or selectors.  Can use more open format


Create a new pod with two labels
```
$ k run hello-world --image=nginx \
--restart=Never -labels=tier=backend,env=dev
```

```
$ k get pods --show-labels
$ k get pods -oyaml | grep -C 2 labels:
```

Set a label on a running pod:
```
$ k label pod hello-world color=blue
$ k label pod hello-world env=inactive --overwrite
```

Remove a label:
```
$ k label pod hello-world color-
```

#### Label Selectors

Equality-based requirement: =, ==, !=.   You can also separate multiple terms with a comma, and combine them with AND

Set-based requirement:  in, notin, exists


```
$ k get pods -l env=prod
```

example - get pods that match label blue or green, and env = prod
```
$ k get pods -l 'color in (blue, green)', env=prod
```

Label selection in a manifest:
```
apiVersion: network.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: hello-world-netpol
spec:
  podSelector:
    matchLabels:
      tier: frontend
```

#### Annotations


modifying live object:
```
$ k annotate pod hello-world release="rel-3-projectname-builddefname"
$ k annotate pod hello-world release="rel-3-projectname-builddefname" --overwrite
```


remove:
```
$ k annotate pod hello-world release-
```

example manifest:

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
  labels: 
    app: hello-world
    env: prod
  annotations:
    commit: '123456abc'
    contact: 'eric@ericlazar.com'
    release: 'rel-2-projectname-builddefname'
    description: "This pod exists to help study for the CKAD exam"
spec:
  containers:
  - name: hello-world
    image: eric/java-hello-world:1.0.0
    env:
    - name: DNS_DOMAIN
      value: cluster
    ports: 
    - containerPort: 8080
  restartPolicy: Never

```

```
$ k get pods hello-world -oyaml | grep -C 5 annotations:
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

### Retained Job History

spec.successfulJobsHistoryLimit
spec.failedJobsHistoryLimit

default is to keep the last 3


## Understand multi-container Pod design patterns (e.g. sidecar, init and others)


### Sidecar Pattern

Container that providers helper functionality to the primary app/container.  It's job is to extend functionality of the main container.

Defined as a second container in the delpoyment manifest

can access volumes that the other containers within the pod can access

### Init Containers

Initialization logic to be run before the main application even starts.

In an init container fails, the whole pod is restarted, causing all init containers to run again.

spec.initContainers



### Adapter Pattern

[kubernetes.io blog](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/#example-3-adapter-containers)

Adapter containers standardize and normalize output




### Ambassador Pattern

An container that proxies the network connection to the main container.

The goal it to hide and/or abstract the complexity of interacting with other parts of the system.


## Utilize persistent and ephemeral volumes


### Persistent Volumes

[kubernetes.io Doc](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

Two persistent storage resources:
- PersistentVolume (PV): storage in the cluster that has been provisioned by an admin or dynamically provisioned using Storage Classes.
- PersistentVolumeClaims (PVC): request for storage by a user.  Can request specific size and access modes (ReadWriteOnce, ReadOnlyMany, ReadWriteMany)


#### Lifecycle of a Volume and Claim

- Provisioning
-- Static
-- Dynamic - requires DefaultStorageClass admission controller.

- Binding - User creates PVC with a specific amt of storage/access mode.  A control loop in the most watches for new PVCs, finds a matching PV (if possible), and binds them together.  A PVC to PV binding is a 1-to-1 mapping, using a **ClaimRef**.   Claims will remain unbound if a matching volume does not exist.

- Using.  Pods use claims as volumes. Pods access their claimed PVs by including a **persistentVolumeClaim** secion on a Pod's **volumes** block.

- Storagse Object in Use Protection - ensure that PVCs in active use by a Pod and PV are not removed.  If a user deletes a PVC in active use by a pod, the PVC is not removed immediately.  PVC removal is postponed until the PVC is no longer actively used by and Pods.  If an admin deleted a PC that is bound to a PVC, the PV is not removed immeditately.  PV removal is postposed until the PV is no longer bound to a PVC.
You can see that a PVC is protected when the PVC's status is **Terminating** and the **Finalizers** list includes **kubernetes.io/pvc-protection**

- Reclaiming

- Retain

- Delete

- Recycle

- PersistentVolume deletion protection finalizer

- Reserving a PersistentVolume

- Expanding Persistent Volume Claims

-- Support for PVCs is enabled by default.  k8s 1.24+ (stable)
-- You can only expand a PVC if its storage class's **allowVolumeExpansion** field is set to true.
-- To request a larger volume for a PVC, edit the **PVC** object (**not the PV**) and specify a larger size.
-- You can only resize volumes containing a file system if the file system is XFS, Ext3, Ext4.









