# Application Environment, Configuration and Security

## Discover and use resources that extend Kubernetes (CRD)

## Understand authentication, authorization and admission control

## Understanding and defining resource requirements, limits and quotas

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


## Create & consume Secrets

3 types of Secrets:
- generic - create a secreate from a file, directory or literal value
- docker-registry - for use with authenticating to a docker registry
- tls - for tls/ssl keys



## Understand ServiceAccounts

## Understand SecurityContexts

