# DEMO

## Pre-demo steps

1. Install `kubectl` tool for your OS.

2. Install any kubernetes solution you want. I'm going to use `minikube` as an example.

3. Enable ingress addon, it will allow you to proceed with ingress part successfully.

```bash
minikube addons enable ingress
```

## DEMO 1

4. Let's deploy pod, rs, deployment demo

```bash
# if you want to deploy in specific namespace
# use -n <namespace_name> option
kubectl apply -f demo1/pod.yaml

# expected result
kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          14s

# describe application
# po stands for pods
kubectl describe po nginx

# check application logs
kubectl logs -f nginx

# delete one pod, what's happen?
kubectl delete po nginx

# let's move one
kubectl apply -f demo1/rs.yaml
kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
nginx-rs-88bpx   1/1     Running   0          71s
nginx-rs-gwhgs   1/1     Running   0          71s
nginx-rs-wzg4q   1/1     Running   0          71s

# nice, let's describe replica set object
# rs stands for replicaset
kubectl describe rs nginx-rs

# pay attention to the following lines:
# we can see current state of our replicas
Annotations:  Replicas:  3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed

# let's delete one of the pods, replace XXXX to one of your pods
kubectl delete po nginx-rs-XXXXX
kubectl get po
NAME             READY   STATUS    RESTARTS   AGE
nginx-rs-gwhgs   1/1     Running   0          3m35s
nginx-rs-rwm5c   1/1     Running   0          32s
nginx-rs-wzg4q   1/1     Running   0          3m35s

# here we are, there is one new pods which replaces deleted
# try to delete replicaset
kubectl delete rs nginx-rs
kubectl get pods
NAME             READY   STATUS        RESTARTS   AGE
nginx-rs-gwhgs   0/1     Terminating   0          4m19s
nginx-rs-wzg4q   0/1     Terminating   0          4m19s

# you can catch terminating state of pods ^
# the final one is Deployment, deploy it by yourself as we've done before
NAME                     READY   STATUS    RESTARTS   AGE
nginx-75b6496574-846x9   1/1     Running   0          67s
nginx-75b6496574-8p7v7   1/1     Running   0          47s
nginx-75b6496574-bbk9s   1/1     Running   0          58s

# let's describe deployment!
# deploy stands for deployment
# looks familiar, isn't it?
kubectl describe deploy nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
# let's test how it will be deployed if we change variable to LOG_LEVEL=error
nginx-7b9887855f-pzhhb   0/1     PodInitializing   0          4s
# new pods should be greated, but why only one?
# describe deployment again
RollingUpdateStrategy:  25% max unavailable, 25% max surge
# the default update strategy is Rolling. it will update by 25 % and terminate by 25%
# for us it will round up update count: 3 / 4 ~ 1, and termination round down: 3 / 4 ~ 0
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  3m10s  deployment-controller  Scaled up replica set nginx-75b6496574 to 3
  Normal  ScalingReplicaSet  70s    deployment-controller  Scaled up replica set nginx-7b9887855f to 1
  Normal  ScalingReplicaSet  64s    deployment-controller  Scaled down replica set nginx-75b6496574 to 2
  Normal  ScalingReplicaSet  64s    deployment-controller  Scaled up replica set nginx-7b9887855f to 2
  Normal  ScalingReplicaSet  58s    deployment-controller  Scaled down replica set nginx-75b6496574 to 1
  Normal  ScalingReplicaSet  57s    deployment-controller  Scaled up replica set nginx-7b9887855f to 3
  Normal  ScalingReplicaSet  45s    deployment-controller  Scaled down replica set nginx-75b6496574 to 0
# we can check how it was describe our deployment again ^
```

## DEMO 2

5. Let's deploy service and ingress and play around it.

```bash
kubectl apply -f demo2/service.yaml
# get svc list; svc stands for Service
kubectl get svc
nginx        ClusterIP   10.99.190.198   <none>        8080/TCP   15s
# nice, let's access to our service from one the containers:
kubectl exec -ti nginx-XXXX -- sh
apt install iputils-ping net-tools
nslookup nginx
Name:      nginx
Address 1: 10.99.190.198 nginx.default.svc.cluster.local

ping nginx
PING nginx.default.svc.cluster.local (10.99.190.198) 56(84) bytes of data.

# and from our PC:
# <LOCAL_PORT>:<PORT_IN_SERVICE>
kubectl port-forward svc/nginx 8000:8080

# navigate into other terminal window and perform
curl 127.0.0.1:8000
# or from browser window

# final part. Ingress!
kubectl apply -f demo2/ingress.yaml
kubectl get ingress
NAME    HOSTS   ADDRESS         PORTS   AGE
nginx   *       192.168.64.15   80      35s

# make curl into IP address below ADDRESS column with /web at the end
curl 192.168.64.15/web

# you should hit into nginx main page =)

### CONGRATULATIONS!
```
