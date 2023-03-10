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

you can test a network policy by creating an ad hoc container and trying to reach a blocked port:

```
$ k run testcontainer --rm -it image=busybox --restart=Never -- /bin/sh
# wget --spider --timeout=1 10.0.0.11

```

## Provide and troubleshoot access to applications via services





## Use Ingress rules to expose applications

