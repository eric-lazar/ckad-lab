# Services And Networking

## Demonstrate basic understanding of NetworkPolicies

Network policies won't have an effect unless you have a network policy controller that supports it, like Cilium, Flannel or NSX.


```
apiVersion: network.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow
spec:
  podSelector:
    matchLabels:
      app: payment-processor
      role: api
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: coffeeshop
```

### Isolating All Pods in a Namespace

```
apiVersion: network.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}       # Apply to all pods
  policyTypes:
  - Ingress
  - Egress
```

You can test a network policy by creating an ad hoc container and trying to reach a blocked port:

```
$ k run testcontainer --rm -it image=busybox --restart=Never -- /bin/sh
# wget --spider --timeout=1 10.0.0.11

```

## Provide and troubleshoot access to applications via services

### Service Types

- ClusterIP - Exposes the Service on a cluster-internal IP.   Only reachable from within the cluster.

- NodePort - Exposes service on worker nodes IP address using a static port.  Accessible from outside the cluster.

- LoadBalancer - Exposes service to an IP that's accessible outside the cluster.

- ExternalName - Maps a service to a DNS name.


```
$ k create service clusterip nginx-service --tcp:80:80
```

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx-service
  ports:
  - port: 80
    targetPort: 80
```


Create a pod and service (clusterIP) with one command:
```
$ k run nginx --image=nginx --restart=Never --port=80 --expose
```

### Proxy

Establish connection to API server via custom port.  THis process will stay running until you break out of it:
```
$ k proxy --port=9000
```


### NodePort

Worker Node port range is 30000 - 32767
The port is assigned automatically

Update a service and change type to NodePort:
```
$ k patch service nginx -p '{ "spec": {"type": "NodePort"} }'
```

If port is listed as 80:32000/TCP, then the service is exposed on port 32000 on the node




## Use Ingress rules to expose applications

[Kubernetes.io Ingress Doc](https://kubernetes.io/docs/concepts/services-networking/ingress/)

Ingress gives Services:
- Externally reachable URLs
- SSL/TLS
- Name based virtual hosting
- Load Balancing

Typically only used for HTTP/HTTPS traffic

To use an Ingress, you need an Ingress Controller


```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example       # If you omit this, a default should be established
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80
```

Ingresses often use annotations to set options like rewrite-target


Each HTTP rule contains the following:
- Optional host.   If no host is specified, then it applied to all inbound HTTP traffic.
- List of Paths.  i.e. /webapp2.  each path has an associated service.  one of these:
-- service.name
-- service.port.name
-- service.port.number
- Backend is combo of Service and Port Names.

### Default Backend

An ingress with no rules sends all traffic to a single default backend, specified in .spec.defaultBackend.  This is typically defined on the Ingress **Controller**.








