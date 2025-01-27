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

Docker will run the container and imediately stop the container this is becuase containers are not meant to run operating systems rather they are use to run processes (web server, application etc.). When we run ubuntu as docker container the dockerfile list the `bash` as the `CMD` which is a shell that looks for a terminal, if no terminal is found  bash will exit

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

this approch is not very good, because the image already conveys that the container will sleep , then why are we specifying the sleep command again ? We can fix this by using `ENTRYPOINT` instruction in image file.

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

now we can run the container without speciying the sleep time

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

How this entrypoint and command maps to kubenetes Pod ? well this is quite straight forward

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
When we have many Pod definition files, then it become difficult to manage the environment data inside the Pod definition files, instead we can store the configuration seperately in config maps.

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
     name: APP_COLOR
     valueFrom:
      configMapKeyRef:
       name: app-config
       key: APP_COLOR
```

## Secrets

if you want to store senstive data as a configuration then use secrets instead of config map. Secret value is stored in base64 format.
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
--from-literal=<key>=<value-in-base64> \
--from-literal=<key>=<value-in-base64>
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
     secretRef:
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
     name: APP_COLOR
     valueFrom:
      secretKeyRef:
       name: app-secret
       key: APP_COLOR
```

### How to inject whole secret as a file in volumes ?

If we inject a secret as a volume in a Pod, then every entry in secret will be stored as a seperate file in the volume.
So if we create `app-secret`

```yaml
volumes:
 - name: app-secret-volume
   secret:
    secretName: app-secret
```