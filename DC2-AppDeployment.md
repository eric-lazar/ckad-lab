# Application Deployment

## Use Kubernetes primitives to implement common deployment strategies (e.g. blue/green or canary)

### Deployments

Deployment -> ReplicaSet -> Pods

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-deployment
  labels:
    app: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
       —name: nginx
          image: nginx:1.23.0
          ports:
           —containerPort: 80
```

```
$ k get deployments
$ k describe deployment hello-world-deployment
```

Deployments keep record of deployment history.  Default is last 10

```
$ k rollout history deployment hello-world-deployment
$ k rollout history deployment hello-world-deployment --revision=2
```

Update image via new deployment yaml, or update directly on live object:
```
$ k get image deployment hello-world-deployment nginx=nginx:1.23.1
```

### Rolling Back Deployment

```
$ k rollout undo deployment hello-world-deployment --to-revision=1
```

This will copy revision 1 as a new revision number 3 (if you went from 2 to 1)


### Scaling a Deployment

Manually scaling a deployment:
```
$ k scale deployment hello-world-deployment --replicas=3
```

Autoscaling:

- HPA - Horizontal Pod Autoscaler - Scale based on CPU and mem thresholds.  This is a standard feature of kubernetes.
- VPA - Vertical Pod Autoscaler - Scales CPU and mem for existing pods based on historic metrics.   This is typically a feature of a cloud provider.


HPA

```
$ k autoscale deployment hello-world-deployment --cpu-precent=70 --min=2 --max=8
$ k get hpa
```

If you see unknown for the current CPU consumption, that usually means the metrics server is not working, misconfigured, or the deployment doesn't specify any resource requirements

spec: autoscaling/v1 only supports HPA based on CPU
autoscaling/v2 support memory and custom metrics




## Understand Deployments and how to perform rolling updates

## Use the Helm package manager to deploy existing packages


