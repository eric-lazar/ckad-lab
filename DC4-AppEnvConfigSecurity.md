# Application Environment, Configuration and Security

## Discover and use resources that extend Kubernetes (CRD)

## Understand authentication, authorization and admission control



## Understanding and defining resource requirements, limits and quotas

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: hello-world-quota
spec:
  hard:
    pods: 2
    request.cpu: "1"
    requests.memory: 1024m
    limits.cpu: "4"
    limits.memory: 4096m

```


## Understand ConfigMaps

### Create ConfigMaps

Literal Values:
```
$ k create configmap db-config --from-literal=db=staging
```

Single file with environment variables:
```
$ k create configmap db-config --from-env-file=config.env
```

Single file:
```
$ k create configmap db-config --from-file=config.txt
```

Directory containing files:
```
$ k create configmap db-config --from-file=app-config
```

ConfigMap YAML - key value pair as literal values:
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  db_url: jdbc:postgresql://localhost/db
  user: eric
```

### Consuming ConfigMaps

Pod YAML that references ConfigMap.  Injects the values from the configmap as env variables:
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-w-config
spec:
  containers:
  - image: nginx:1.23.0
    name: webapp
    envFrom:
    - configMapRef:
        name: app-config
```

Reassign variables from ConfigMap into different env vars:
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-w-config
spec:
  containers:
  - image: nginx:1.23.0
    name: webapp
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: db_url
```


Mounting a ConfigMap as a volume.   This will put 2 files in /etc/config called db_url and user.
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-w-config
spec:
  containers:
  - image: nginx:1.23.0
    name: webapp
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```


## Create & Consume Secrets


3 types of Secrets:
- generic - create a secreate from a file, directory or literal value
- docker-registry - for use with authenticating to a docker registry
- tls - for tls/ssl keys

### Create Secret

Imperatively create secret:
Literal Values:
```
$ k create secret generic db-pwd --from-literal=pwd=P@ssw0rd^
```

File containing env vars:
```
$ k create secret generic db-pwd --from-env=file=password.env
```

File containng key:
```
$ k create secret generic ssh-key --from-file=id_rsa=~/.ssh/is_rsa
```

When you declaritively create a secret via YAML, you have to base64 encode the values first

```
$ echo -n 'P@ssw0rd^' | base64
UEBzc3cwcmRe
```

```
apiVersion: v1
kind: Secret
metadata:
  name: db-pwd
type: Opaque
data:
  pwd: UEBzc3cwcmRe
```

### Consume Secret

This will make the secret available in an environment variable in the pod not base64 encoded
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-w-secret
spec:
  containers:
  - image: nginx:1.23.0
    name: webapp
    envFrom:
    - secretRef:
        name: db-pwd 
```

Mounting a secret as a volume.   This will create a file in /var/app called db-pwd with the base64-decoded password in it

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-w-secret
spec:
  containers:
  - image: nginx:1.23.0
    name: webapp
    volumeMounts:
    - name: secret-volume
      mountPath: /var/app
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: db-pwd
```


## Understand Service Accounts

Pods use a service account to authenticate wih the API server using an authentication token.  A k8s admin assigns rules to a Service Account via RBAC.

If not assigned specifically a pod uses a default Service Account.  The default Service Account has the same permissions as an authenticated user.



## Understand SecurityContexts

By default, containers run with root privileges.  As a best practice you should run containers with an ID other than 0 (root).
A Security Context defines priviliges and access control settings for a pod or container.
- The user ID to run the pod
- The group ID for filesystem access
- Granting a process in the container some privs of the root user, but not al of them

PodSecurityContext = pod level
SecurityContext = container level (take precedence over pod level)

Container level securityContext:
```
apiVersion: v1
kind: Pod
metadata:
  name: non-root-pod
spec:
  containers:
  - image: nginx:1.23.0
    name: secure-container
    securityContext: 
      runAsNonRoot: true
```

Pod level securityContext.   Files created on the filesystem are set as owner group ID 3500 (an arbitrary group ID):

```
apiVersion: v1
kind: Pod
metadata:
  name: fs-secure
spec:
securityContext: 
    fsGroup: 3500
  containers:
  - image: nginx:1.23.0
    name: secure-container
    volumeMounts:
    - name: data-volume
      mountPath: /data/app-data
  volumes:
  - name: data-volume
    emptyDir: {}
    
```

