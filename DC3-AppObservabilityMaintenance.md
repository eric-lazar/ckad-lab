# Application Observability and Maintenance

## Understand API deprecations

[kubernetes.io Deprecation Policy](https://kubernetes.io/docs/reference/using-api/deprecation-policy/)

- Alpha (experimental)
- Beta (pre-release)
- GA (generally available)

Rules are enforced between official releases:
- API elements may only be removed by incrementing the version of the API group.
- API object must be able to round-trip between API versions in a given release without information loss.  ex:  object can be written in v1 and read in v2
- An API version in a given track may not be deprecated in favor of a less stable API version
- API lifetime is determined by the API stability level
- The "preferred" API version and the "storage version" for a given group may not advance until after a release has been makde that supports both the new version and the previous version.



## Implement probes and health checks

- Readiness Probe - is the app ready after it starts?
- Liveness Probe - once it's running, is it still OK? if not, restart
- Startup Probe - wait period before starting readiness probe.  If it doesn't start within a certain timeframe, restart container

### Health Verification Methods

- Custom Command - exec.command - executes command in container
- HTTP GET request - httpGet - 200-399 is success, anything else is error
- TCP socket connection - tcpSocket - success on connect

Attributed for fine-tuning health check behavior
- initialDelaySeconds - default: 0 - delay before first check
- periodSeconds - default: 10 - every X seconds
- timeoutSeconds - default: 1 - secs until timeout
- successThreshold - default: 1 - # of successful attempt before it's considered OK
- failureThredhols - default: 3 - after this many failures, probe takes action (restart, etc)



### Readiness Probe

This is the most important probe.

this example waits 3 seconds, then runs readiness probe every 5 seconds:
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
      name: web-port
    readinessProbe:
      httpGet:
        path: /
        port: web-port
      initialDelationSeconds: 3
      periodSeconds: 5
  restartPolicy: Never

```

### Liveness Probe

example from kubernetes.io:
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600       # healthy for first 30 seconds, then unhealthy
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5        # Wait 5 seconds before first attempt
      periodSeconds: 5              # Check every 5 seconds
```

### Startup Probe

Often used for legacy or slow starting applications

```
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 1
  periodSeconds: 10

startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
```


## Use provided tools to monitor Kubernetes applications



## Utilize container logs

## Debugging in Kubernetes


