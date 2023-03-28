

Resources:
[DevOpsCube Study Guide](https://devopscube.com/ckad-exam-study-guide/)


Create a deployment with the name greatdeploy with 5 replicas and the image nginx:1.14.1

Perform a rolling update of the deployment, to nginx:1.14.2

Do a rollback


get pods IP and use a temp busybox to wget it's IP

```
kubectl get po -o wide 

kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- x.x.x.x:80
```


Create a busybox pod that echoes 'hello world' and then exits
```
kubectl run busybox --image=busybox -it --restart=Never -- echo 'hello world'
```

To do the same but exit afterwards:  add --rm


Create an nginx pod and set an env value as 'var=val'. Check the env value existence within the pod

```
kubectl run nginx --image=nginx --restart=Never --env=var=val

kubectl exec -it nginx -- env
```

Multicontainer pods:
https://github.com/dgkanatsios/CKAD-exercises/blob/main/b.multi_container_pods.md


create config map options with var5=val5
```
kubectl create cm options --from-literal=var5=val5
```



