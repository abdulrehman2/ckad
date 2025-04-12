
# Readiness Probe

## Pod status
 A pod have a pod status and conditions. When we create a pod then it will go through different statuses
 - `Pending`: a pod when created will be in pending state until the scheduler find a node to place it
 - `ContainerCreating`: Once it is placed on a node, the k8s will pull the images to create the pod, this will be the ContainerCreating state
 - `Running`: Once all images are downloaded, the Pod will be in the running state, until it is completed or terminated.

## Pod Conditions
Conditions compliment pod status, it is an array of values that tell us the state of the pod
- `PodScheduled`: once pod is scheduled then this flag will be set to true
- `Initialized`: Once the pod is initialized then this flag will be set to true
- `ContainersReady`: Once all containers inside the Pod are ready then this flag will be true
- `Ready`: Finally the Pod is considered to be ready once all above flags are true,

A pod once get ready might not be able to serve the application users immediately, for example a web server serving the front end might take few more
 seconds/minutes until it is able to respond to the user requests. However the Pod status is ready ,
  how we can make sure that the traffic is not routed to this pod until the Pod is "actually" ready in order to avoid service disruption ?

> We can configure readiness probes for the application. This readiness probe will allow the K8s to determine whether the pod is actually ready or not. 
Once a container is created, k8s will not set the 'ContainersReady' to be true, instead it will run a test (probe) to check if the pod is accessible or not.
 
 - For Http: we can simply expose an endpoint to verify if the application is reachable i.e. /api/ready
 - For TCP: We need to test if a particular TCP socket is listening and available for connections.
 - A custom script: Or we can run a custom script inside the container that will exit successfully to check if the container is ready or not.
 

  How to configure different readiness probes ?

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: my-app
spec:
 containers:
  name: nginx
  image: nginx
  readinessProbe:
   httpGet:
    path: /api/ready
    port: 8080
```

- if you want to specify an initial delay in order for the application to warm we can specify an initial delay using `initialDelaySeconds`.
- If we want to specify how often to probe, we can set `periodSeconds`
- By default if the application is not ready after three attempts the probe will stop, we can increase the threshold by using `failureThreshold` 

 For TCP
```yaml
readinessProbe:
 tcpSocket:
  port: 3306
```

Custom script
```yaml
readinessProbe:
 exec: 
  command:
   - cat
   - /app/is_ready
```

# Liveness Probe
A container might be in ready state and available to serve the users, but there is a possibility that in reality it is not working as expected and will not serve the users. For example, the application
is stuck in a loop or crashed. In order to cover such scenario we have liveness probe. This test will allow the k8s to verify if the application is able to serve the users. We can configure the liveness probe in
similar fashion like we did for readiness probe. The liveness check has to be exposed by the application developer based on the certain requirements that needs to be fulfilled by the application.


# Container Logs

for docker we can get the logs of a container using

```bash
docker logs -f <container-id>
``` 

In kubernetes we can get it using

```bash
kubectl logs -f pod-name
```

if we have multiple containers within the pod then 

```bash
kubectl logs -f pod-name  container-name
```

# Metrics server

We can have one metrics server per k8s cluster
metrics server pulls metrics from all nodes of a cluster and store the information in the memory, hence it an in memory solution. 
The kubelet has a component called `cAdvisor` that will allow to pull the performance metrics of containers and send to the metrics server.

get node metrics

```bash
kubectl top node
```


```bash
kubectl top pod
```