# Application Environment, Configuration and Security

## Discover and use resources that extend Kubernetes (CRD)

CRD = Custom Resource Defintion.  Extensions of the k8s API, that store a collection of API objects.


Custom Resources are combined with Custom Controllers

The Operator pattern combines custom resources and custom controllers

CRDs are stored in the API server, so storing too much data there can overwhelm the API server.






## Understand authentication, authorization and admission control

[kubernetes.io Doc](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)

### API Authentication

Two category of users:   users and service accounts (managed by k8s)


Authentication Methods:

- Username.  string which identifies end user. 
- UID. More unique than a username
- Groups. set of strings which ident user's membership.
- Extra Fields.


### X509 Client Certs

Client cert authe is enabled by passing the --client-ca-file=FILENAME to the API server.   File much contain CA cert.

This creates CSR for user, with two groups (app1 and app2)
```
openssl req -new -key jbeda.pem -out jbeda-csr.pem -subj "/CN=jbeda/O=app1/O=app2"
```

### Static Token File

---token-auth-file=TOKENFILE

Token file is a CSV file with a minimum of 3 columns:  token, usernane, user uid

```
token,user,uid,"group1,group2,group3"
```

Bearer token in request:
```
Authorization: Bearer 31ada4fd-adec-460c-809a-9e56ceb75269
```

Bootstrap token is same format as bearer token

### Service Account Tokens

A service account is an automatically enabled authenticator that uses signed bearer tokens to verify requests

--service-account-key-file - PEM encoded x509 RSA or ECDSA keys.


Create service account and token:
```
$ k create service account adosvcacct
$ k create token adosvcaccount
```

The token created is a signed JSON Web Token (JWT)


### OpenID Connect Tokens

[kuberneter.io Doc](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#openid-connect-tokens)

Used to authenticate to things like Active Directory

example:
```
kubectl config set-credentials USER_NAME \
   --auth-provider=oidc \
   --auth-provider-arg=idp-issuer-url=( issuer url ) \
   --auth-provider-arg=client-id=( your client id ) \
   --auth-provider-arg=client-secret=( your client secret ) \
   --auth-provider-arg=refresh-token=( your refresh token ) \
   --auth-provider-arg=idp-certificate-authority=( path to your ca certificate ) \
   --auth-provider-arg=id-token=( your id_token )
```


Option 1 - OIDC Authenticator

```
users:
- name: mmosley
  user:
    auth-provider:
      config:
        client-id: kubernetes
        client-secret: 1db158f6-177d-4d9c-8a8b-d36869918ec5
        id-token: eyJraWQiOiJDTj1vaWRjaWRwLnRyZW1vbG8ubGFuLCBPVT1EZW1vLCBPPVRybWVvbG8gU2VjdXJpdHksIEw9QXJsaW5ndG9uLCBTVD1WaXJnaW5pYSwgQz1VUy1DTj1rdWJlLWNhLTEyMDIxNDc5MjEwMzYwNzMyMTUyIiwiYWxnIjoiUlMyNTYifQ.eyJpc3MiOiJodHRwczovL29pZGNpZHAudHJlbW9sby5sYW46ODQ0My9hdXRoL2lkcC9P...
        idp-certificate-authority: /root/ca.pem
        idp-issuer-url: https://oidcidp.tremolo.lan:8443/auth/idp/OidcIdP
        refresh-token: q1bKLFOyUiosTfawzA93TzZIDzH2T...
      name: oidc
```

Option 2 - use the -token option
```
kubectl --token=eyJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJodHRwczovL21sYi50cmVtb2xvLmxhbjo4MDQzL2F1dGgvaWRwL29pZGMiLCJhdWQiOiJrdWJlcm5ldGVzIiwiZXhwIjoxNDc0NTk2NjY5LCJqdGkiOiI2RDUzNXoxUEpFNjJOR3QxaWVyYm9RIiwiaWF0IjoxNDc0NTk2MzY5LCJuYmYiOjE0NzQ1OTYyNDksInN1YiI6Im13aW5kdSIsInVzZXJfc... get nodes
```

### Webhook Token Auth

Authenticate to a remote webhook service


example:
```
# Kubernetes API version
apiVersion: v1
# kind of the API object
kind: Config
# clusters refers to the remote service.
clusters:
  - name: name-of-remote-authn-service
    cluster:
      certificate-authority: /path/to/ca.pem         # CA for verifying the remote service.
      server: https://authn.example.com/authenticate # URL of remote service to query. 'https' recommended for production.

# users refers to the API server's webhook configuration.
users:
  - name: name-of-api-server
    user:
      client-certificate: /path/to/cert.pem # cert for the webhook plugin to use
      client-key: /path/to/key.pem          # key matching the cert

# kubeconfig files require a context. Provide one for the API server.
current-context: webhook
contexts:
- context:
    cluster: name-of-remote-authn-service
    user: name-of-api-server
  name: webhook
```

### Authenticating Proxy

[kubernetes.io Doc](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#authenticating-proxy)





### Authorization Overview


[kubernetes.io Doc](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)

k8s authorizes API requests using an API server.  It evaluates request attibutes against all policies and allows or denied the request.


When mutliple auth modules are configured, each is checked in sequence.  If any authorizer approves or denies a request, that decision is immediately returned and no other authorizer is consulted.
If all modules have no opinion, request is denied (403).



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

