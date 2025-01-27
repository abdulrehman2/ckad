# 1. Core Concepts
## PODS
- Pods are an abstraction over the running container in k8s.
- We can run multiple containers inside the same pod but usually that not the case when we want to scale the application, instead we run each container as a seperate POD
- We can run multiple containers inside same pod , but these containers are of different type, for example (helper containers)
Containers inside the same POD will have access to same network space and storage, this will automatically handled by k8s

How to run a POD ?
``` bash
kubectl run nginx --image nginx
```
By default the docker hub is set as default repository for images. We can change that to point to a private repository

How to get running PODS ?

``` bash
kubectl get pods
```
## YAML Basics

Every yaml file for k8s have 4 top level fields i.e.  `apiVersion`, `kind`, `metadata` and `spec`

`kind` can have following values based on the resource type we are creating

| Kind          | Version |
| --------------| ------- |
| POD           | v1      |
| Service       | v1      |
| ReplicaSet    | apps/v1 |
| Deployment    | apps/v1 |

`metadata` allow us to set the name of the pod and labels. We can add many labels as we want that can be used to filter pods later when we scale.

`spec` is the place where we setup the details for the container we want to run. spec is a dictionary.


```yaml

apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
 labels:
  app: myapp
  type: front-end
spec:
 containers:
  - name: nginx-container
    image: nginx
```

to get more detail of running pods we can use

```bash
kubectl get pods -o wide
```

we can get the yaml of a command and save it in a file like this

```bash
kubectl run redis --image=redis123 --dry-run=client -o yaml > redis.yaml
```

to display contents of a file we can use `cat` command

```bash
cat redis.yaml
```

if you are asked to edit an existing POD, please note the following:

 - If you are given a pod definition file, edit that file and use it to create a new pod.

- If you are not given a pod definition file, you may extract the definition to a file using the below command:

``` bash
kubectl get pod <pod-name> -o yaml > pod-definition.yaml
```
Then edit the file to make the necessary changes, delete, and re-create the pod.

To modify the properties of the pod, you can utilize the `kubectl edit pod <pod-name>` command. Please note that only the properties listed below are editable.

- spec.containers[*].image

- spec.initContainers[*].image

- spec.activeDeadlineSeconds

- spec.tolerations

- spec.terminationGracePeriodSeconds



## Kubernetes Replication Controller and Replica Set

A controller is the brain of k8s, it is a set the processes that monitors k8s objects and respond accordingly.

a *replication controller* allow us to make sure to run multiple instances of a POD at any given time thus allowing high availability.

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
 name: myapp-replica
 labels:
  app: myapp
  type: front-end
spec:
 template:
  metadata:
   name: myapp-pod
   labels:
    app: myapp
    type: front-end
  spec:
   containers:
    - name: nginx-container
      image: nginx
 replicas: 3   
```

Difference between `ReplicationController` and `ReplicaSet` is you need to provide `selector` in second case. This selector will allow not only to monitor new pods created by replica set but also monitor existing pods with same matching labels

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
 name: myapp-replica-set
 labels:
  app: myapp
  type: front-end
spec:
 template:
  metadata:
   name: myapp-pod
   labels:
    app: myapp
    type: front-end
  spec:
   containers:
    - name: nginx-container
      image: nginx
 replicas: 3
 selector:
  matchLabels:
   type: front-end
```

To change number of replicas you can edit the definition file and use the `replace` command

```bash
kubectl replace -f replica-set-demo.yml
```

to manually scale the replicate set you use `scale` command. You can either give the name of the file or the name of the replica set , but keep in mind this will not alter the file, it will just update the number of replicas 

```bash
kubectl scale --replicas=3 replicaset my-app-replica-set
# or
kubectl scale --replicas=3 -f replica-set-demo.yml
```

## Namespaces

By default k8s create `default`, `kube-system`  namespaces.
 - POD within the same namespace can access services by refering just name of service
 - POD from one namespace can access service in another namespace by refering this structure `service-name.namespace-name.svc.local.cluster`

to get pods in a different namespace other than default we use

```bash
kubectl get pods --namespace=dev
```
how to create a namespace ?

```yaml
apiVersion: v1
kind: Namespace
metadata:
 name: dev
```

```bash
kubectl create namespace dev
# or
kubectl create -f namespace-dev.yml
```

How to switch to dev namespace for current session ?

```bash
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

we can assign resource quota to each namespaces for example

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
 name: my-quota
 namespace: dev
spec:
 hard:
  pods: "10",

```

## Imperative Commands


```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml

kubectl create deployment nginx --image=nginx --replicas=3 --dry-run=client -o yaml > nginx-deployment.yaml

# this will automatically use the redis pod labels as selector 
kubectl expose pod redis --port=6379 --name=redis-service --dry-run=client -o yaml 

# or # this will not use the redis pod labels as selector instead  it will assume selectors as app=redis
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml

# create a pod with custom port
kubectl run custom-nginx --image=nginx --port=8080

## Create a pod and expose it in a single command
## *when you use expose command then port is required
kubectl run httpd --image=httpd:alpine --expose=true --port=80

```