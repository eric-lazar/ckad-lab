# Application Observability and Maintenance

## Understand API deprecations

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



### Startup Probe




## Use provided tools to monitor Kubernetes applications

## Utilize container logs

## Debugging in Kubernetes


