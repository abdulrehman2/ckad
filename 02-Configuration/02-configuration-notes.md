## Docker images

#### Basics of a DockerFile !

 - Dockerfile is constructed in `Instruction` and `Argument` format, every Dockerfile start with the `FROM` instruction

- Every docker image is based on some other base image

- When the image is run as a container, the `ENTRYPOINT` command will be executed, hence we need to specify this command in order to run a application.

-- Docker build images in a layer structure, every instruction will create a new layer

```DockerFile
FROM Ubuntu

RUN ap-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

```bash
# how to build an image
docker build Dockerfile -t abdulrehman/my-custom-app

#how to push the image
docker push abdulrehman/my-custom-app 
```

What will happen if run `docker run ubuntu` ?

Docker will run the container and immediately stop the container this is because containers are not meant to run operating systems rather they are use to run processes (web server, application etc.). When we run ubuntu as docker container the dockerfile list the `bash` as the `CMD` which is a shell that looks for a terminal, if no terminal is found  bash will exit

what if we change the command while running the container

```bash
docker run ubuntu sleep 5
```
docker will run the container, execute the sleep process and then exit.
We can create our own docker image to make sure container always run the container with sleep command

```Dockerfile
FROM Ubuntu

CMD sleep 5
```
and we will build it using

```bash
docker build -t ubuntu-sleep .
```

there are different formats for the `CMD` instruction

```Dockerfile
CMD command param1
CMD ["command","param1"]
```
the first format is shell format, the second format is the JSON array format, in this case make sure the first command is executable  i.e. `sleep`

What if we want to change the number of seconds for the sleep command ?  we can modify the `run` command

```bash
docker run ubuntu-sleeper sleep 10
```

this approach is not very good, because the image already conveys that the container will sleep , then why are we specifying the sleep command again ? We can fix this by using `ENTRYPOINT` instruction in image file.

```Dockerfile
FROM Ubuntu
ENTRYPOINT ["sleep"]
```

then we can do something like this

```bash
docker run ubuntu-sleeper 10
```

Difference between `CMD` and `ENTRYPOINT` is that the in case of `CMD` , the command specified in the run command will replace the command specified in the image, while in case of `ENTRYPOINT` the command will be appended with the existing command in image file.

What if we want to specify the default value for sleep, instead of passing it every time ? then we can combine the `CMD` and `ENTRYPOINT`

```Dockerfile
FROM Ubuntu

ENTRYPOINT ["sleep"]

CMD ["5"]
```

now we can run the container without specifying the sleep time

```bash
docker run ubuntu-sleeper
```

if we want to change the sleep time we can use simply

```bash
docker run ubuntu-sleeper 10
```

What if we want to replace the entrypoint at the runtime ?

```bash
docker run --entrypoint sleep2.0 ubuntu-sleeper 10
```

How this entrypoint and command maps to kubernetes Pod ? well this is quite straight forward

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: ubuntu-sleeper-pod
spec:
 containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    command: ["sleep2.0"]   # command maps to ENTRYPOINT instruction of docker ENTRYPOINT ["sleep"]
    args : ["10"]           # args maps to the CMD instruction of docker CMD ["5"]
```

> Keep in mind you can only edit certain properties of a running pod, following properties can edited at runtime

```bash
kubectl edit pod <pod name>
```

```yaml
spec.containers[*].image
spec.initContainers[*].image
spec.activeDeadlineSeconds
spec.tolerations
```

While for a deployment any field can be modify it. Since Pod template is a child of deployment, every modification to Pod will automatically result in deletion of existing pod and creation of new one automatically.

```bash
kubectl edit deployment my-deployment
```

if you want to replace an existing pod with new definition we can use the `replace` command. This will first delete the existing pod and replace it with new one


```bash
kubectl replace --force -f new-pod-definition.yaml
```

## Config Maps
When we have many Pod definition files, then it become difficult to manage the environment data inside the Pod definition files, instead we can store the configuration separately in config maps.

How to create one using imperative way ?

```bash
kubectl create configmap <config-map-name> \
--from-literal=<key>=<value> \
--from-literal=<key>=<value> \
```
What if you want to create many items in config map ! this become cumbersome to add them manually. We can specify a file having the key value pairs

```bash
kubectl create configmap <config-map-name> --from-file=<path-to-file>

#example
kubectl create configmap app-config --from-file=app_config.properties
```
Declarative way ?

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
 name: app-config
data:
 APP_COLOR: blue
 APP_MODE: prod
```
then run 

```bash
kubectl create -f config-map.yaml
```

### How to inject the config map in Pod definition ? 

> Refer complete config map

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: web-app
 labels: 
  app: web-app
spec:
 containers:
  - image: web-app-color
    name: web-app-color
    envFrom:
     - configMapRef:
        name: app-config
```

> Refer a single item from config map ?

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: web-app
 labels: 
  app: web-app
spec:
 containers:
  - image: web-app-color
    name: web-app-color
    env:
     - name: APP_COLOR
       valueFrom:
        configMapKeyRef:
         name: app-config
         key: APP_COLOR
```

## Secrets

if you want to store sensitive data as a configuration then use secrets instead of config map. Secret value is stored in base64 format.
We can use following commands to encode/decode a string to base44

```bash
#this will encode the string my-username (bXktdXNlcm5hbWU=)
echo -n 'my-username' | base64

# this will decode the string
echo -n 'bXktdXNlcm5hbWU=' | base64 --decode
```

### How to create a secret in imperative way ?

```bash
kubectl create secret generic <secret-name> \
--from-literal=<key>=<value> \
--from-literal=<key>=<value>
```
### What if you want to pick key-values from a file ?

```bash
kubectl create secret generic <secret-name> --from-file=<path-to-file>

#example
kubectl create secret generic app-secret --from-file=app_secrets.properties
```

### Declarative way ?

```yaml
apiVersion: v1
kind: Secret
metadata:
 name: app-secret
data:
 DB_USER: Ymx1ZQ== #blue
 DB_PASSWORD: cHJvZA== #prod
```

### How to use the refer complete secret in pod definition ?

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: webapp
spec:
 containers:
  - image: webapp-image
    name: webapp
    envFrom:
     - secretRef:
        name: app-secret
```


### How to  refer only a single key from secret in pod definition ?

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: webapp
spec:
 containers:
  - image: webapp-image
    name: webapp
    env:
     - name: APP_COLOR
       valueFrom:
        secretKeyRef:
         name: app-secret
         key: APP_COLOR
```

### How to inject whole secret as a file in volumes ?

If we inject a secret as a volume in a Pod, then every entry in secret will be stored as a separate file in the volume.
So if we create `app-secret`

```yaml
volumes:
 - name: app-secret-volume
   secret:
    secretName: app-secret
```

## Encrypt Secrets at rest 
By default secrets in etcd are not encrypted.

How to view data at rest in etcd ?

```bash
ETCDCTL_API=3 etcdctl \
--cacert=/etc/kubernetes/pki/etcd/ca.crt   \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key  \
get /registry/secrets/default/secret1 | hexdump -C
```


## Docker Security 
- Containers and host share the same kernel they are not completely isolated.
- They are isolated by using different namespaces within linux.
- By default a container run using a root user, this root user is different in terms of capabilities it has as compare to the root user of the host.
- We can change the user of a container by either specifying it in the image file or with run command `--user=1000`

## Security Context
We can configure security context at Pod level or container level
- If security context is set at pod level it will be applicable to all containers inside the pod
- If we configure the security context both at Pod and container, then container context will override the Pod context


### Setting context at Pod level
```yaml
apiVersion: v1
kind: Pod
metadata:
 name : ubuntu-pod
securityContext:
     runAsUser: 1000
     capabilities:
      add: ['MAC_ADMIN']
spec:
 containers:
  - image: ubuntu
    name: ubuntu
```

### Setting context at container level
```yaml
apiVersion: v1
kind: Pod
metadata:
 name : ubuntu-pod
spec:
 containers:
  - image: ubuntu
    name: ubuntu
    securityContext:
     runAsUser: 1000
     capabilities:
      add: ['MAC_ADMIN']
```

```bash
# how to get the name of user with which pod is running under
kubectl exec ubuntu-sleeper -- whoami
```

## Service Accounts

Service accounts are used by different services to access kubernetes cluster. We can control access to kubernetes cluster using Role based access control. 
 - For example Prometheus access the k8s performance metrics using the service account
- If an application hosted outside of a k8s cluster, and wanted to access some information from k8s (for eg. get list of pods) then we create a service account for this
and k8s will create a relevant token (JWT) for this account hold as a secret that will authenticate while accessing k8s API's
- If an application is hosted inside the k8s then we mount the token secret as a volume that can be used by the application inside the Pod

```bash
# we can get the contents of a token by running
kubectl exec -it my-dashboard-pod cat /var/run/secrets/kubernetes.io/serviceaccount
```

How to set the custom service account ?
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: my-dashboard-pod
spec:
 containers:
  - name: my-kubernetes-dashboard
    image: my-kubernetes-dashboard
serviceAccountName: dashboard-sa
#if you do not want to automount the default service account then use
automountServiceAccountToken: false
```


- By default, every namespace has a service account with name `default`, this account secret is mounted as a volume automatically with every Pod we create inside the namespace. The default account has very limited access like running queries on k8s.
- Before k8s 1.22 there was no expiration date/time of token and it is not bound to any specific audience.
- In 1.22 `TokenRequestAPI` was introduced an API that will allow to provision tokens for service accounts, these tokens are (audience, time and object bound) hence more secure When a new Pod is created, a new token is generated using the TokenRequestAPI (via service account admission controller) and this token is mounted as a projected volume to the Pod.

```yaml

metadata:
 name: nginx
spec:
 containers:
  - name: nginx
    image: nginx
    volumeMounts:
     - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
       name: kube-api-acccess-6mtg8
       readOnly: true
 volumes:
  - name: kube-api-acccess-6mtg8
    projected:
     defaultMode: 420
     sources:
       - serviceAccountToken:
           expirationSeconds: 3607
           path: token
           
```

- In 1.24, no longer a secret is created, with the creation of service account, instead if you want to create one you can do that explicitly.

```bash
kubectl create token service-account-name
```
 or you can create a yaml

 ```yaml
apiVersion: vi
kind: Secret
type: kubernetes.io/service-account-token
matedata:
  name: secretname
  annotations:
   kubernetes.io/service-account.name: dashboard-sa

 ```

 ## Resource Requirements
 When we create a Pod, we can specify the minimum amount of memory and compute requirements for a a specific container. The scheduler is responsible to determine the best node that can handle these resource requirements and then create a Pod.

 ```yaml
apiVersion: v1
kind: Pod
metadata:
 name : ubuntu-pod
spec:
 containers:
  - image: ubuntu
    name: ubuntu
    resources:
     requests:
      memory: '4Gi'
      cpu: 1
 ```

 ### CPU units
- 1 cpu mean different based on the environment
  - 1 = Azure Core
  - 1 = GCP Core
  - 1 = AWS vCPU
  - 1 = Hyper thread
- We can specify the compute/cpu as 1, 0.1 or 100m where `m` stands for mili. we can go as low as 1m but not lower than that. 


### Memory units
- Kubernetes has 2 units supported for memory
- Note the difference between `G` and `Gi`

|Type|Unit|Value|
|--|--|--|
|1 Gigabyte|G|1,000,000,000 bytes|
|1 Megabyte|M|1,000,000 bytes|
|1 kilobyte|K|1,000 bytes|
|--|--|--|
|1 Gibibyte|Gi|1,073,741,824 bytes|
|1 Mebibyte|Mi|1,048,576 bytes|
|1 kibibyte|Ki|1,024 bytes|

By default, the container has no restrictions on how much memory it can consume, we can limit this by using `limits`

 ```yaml
apiVersion: v1
kind: Pod
metadata:
 name : ubuntu-pod
spec:
 containers:
  - image: ubuntu
    name: ubuntu
    resources:
     requests:
      memory: '1Gi'
      cpu: 1
     limits:
      memory: '2Gi'
      cpu: 2
 ```

 ** When a pod tries to exceed the cpu limit, then system will throttle the cpu, so that it cannot consume more than that, however in case of memory, the POD can use more memory than it is mentioned in limits, but if it does it for a longer period of time, then k8s will terminate the Pod with OOM (Out of Memory) error.


###  Request/Limit Behavior

 #### No Request and Limit Set
 In this case one Pod may consume all the resources, while other Pod will have no resources left.

 #### No Request but Limit Set
 In this case , limits will be considered as the request i.e. (request=limit), this will make sure all pods will get at least the requested resources

 #### Request and Limit Set
 In this case, Pods will get the requested resources, but can't consume more than the limits. The problem with this approach is that if one Pod needs more cpu cycles than the limit and these cycles are already available (since other Pod is not consuming it), then the Pod will under perform. 

#### Request set but not Limit
 In this case, Pods will get the requested resources, and they can consume, hence resources will not be under used.

 ## How to set default limits
 We can set a configuration to define the default limits/request for all new containers. Changing these limits will not affect existing pods rather only new one.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
 name: cpu-resource-constraint
spec:
 limits:
  - default:
     cpu: 500m      #limit
    defaultRequest:
     cpu: 500m      #request
    max:
     cpu: "1"       #limit
    min:
     cpu: 100m      #request
    type: Container
 ```

```yaml
apiVersion: v1
kind: LimitRange
metadata:
 name: memory-resource-constraint
spec:
 limits:
  - default:
     memory: 1Gi      #limit
    defaultRequest:
     memory: 1Gi      #request
    max:
     memory: 1Gi       #limit
    min:
     memory: 500Mi      #request
    type: Container
 ```

 ## Resource Quota
We can set resource quota at namespace level, so that any given namespace should not consume more than the limits

  ``` yaml
apiVersion: v1
kind: ResourceQuota
metadata:
 name: my-resource-quota
spec:
 hard:
   requests.cpu: 4
   requests.memory: 4Gi
   limits.cpu: 10
   limits.memory:10Gi
 ```

## Taints and Toleration

- If we want to make sure that some nodes should not be used by certain Pods, 
  then we add taint (repellant) to that particular nodes. By default Pods are not tolerant to taints.
  Meaning a node that has some taint will not allow to schedule any Pod that is not tolerated for that particular taint.
- If we want to make sure that only certain pod can be scheduled on a node,
  we first add the taint on the node and we make that pod tolerant to that particular taint,
  in this way only certain pods can be placed on certain nodes.

> Taints are set on nodes, while toleration are set on Pods

### How to taint a node ?

```bash
kubectl taint nodes node-name key=value:taint-effect

#example if we want to taint a node for Pods having key value pair like app:blue then

kubectl taint nodes node-name app=blue:NoSchedule
```

taint effects can be  
 - NoSchedule (will not schedule pods that are not tolerant)
 - PreferNoSchedule (will try not to schedule pods that are not tolerant but not guaranteed)
 - NoExecute (no schedule for new pods and existing pods will be terminated if not tolerate the taint)


### How to tolerate a Pod ?
add tolerations section under spec, and make sure to quote the values for key, operator, value and effect
```yaml
apiVersion: v1
kind: Pod
metadata:
 name : ubuntu-pod
spec:
 containers:
  - image: ubuntu
    name: ubuntu
  tolerations:
   - key: "app"
     operator: "Equal"
     value: "blue"
     effect: "NoSchedule" 
```

## Node Selector
 - When we want that a certain pod should always be scheduled on a certain node, than we can set the node selector for each pod.
 - We first have to add a label to the node, then we will mention it as a selector in pod definition

```bash
kubectl label node node-1 size=Large
```

after this we can add the selector in pod

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: data-processor
spec:
 containers:
  - name: data-processor
    image: data-processor
 nodeSelector:
  size: Large
```
> Node selectors has limitations, we only use a single label and selector to achieve our goal.
 What if we have a requirement where we want to place the pod in medium or large nodes, or we want to place pods on any node other than small nodes. To solve this problem we have **node affinity**.

 ## Node Affinity


 ```yaml
apiVersion: v1
kind: Pod
metadata:
 name: data-processor
spec:
 containers:
  - image: data-processor
    name: data-processor
 affinity:
  nodeAffinity:
   requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
     - matchExpressions:
       - key: size
         operator: In
         values:
          - Large
          - Medium
 ```

 There are many operators available for example
- `Exists` operator will make sure that the a particular label exists on a node, no need to check the value
- `NotIn` operator will be opposite of `In`

What will happen if the Pod does not find any node with given label or the label is changed in the future ? To answer this question we have to consider the 

 - requiredDuringSchedulingIgnoredDuringExecution
    - if no node is found with matching label then the pod will not be scheduled.
 - preferredDuringSchedulingIgnoredDuringExecution
    - k8s will try to schedule the pod on a node with matching label, however if not found Pod will be scheduled on some other node.


|Type|DuringScheduling|DuringExecution|
|----|----------------|---------------|
|Type 1| Required|Ignored|
|Type 2| Preferred|Ignored|
