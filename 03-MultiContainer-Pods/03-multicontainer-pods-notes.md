# Multi Container Pods

## Init Containers

Sometimes you might want to run some task or process before an application can start, like running the database migrations and it should be done only once when the Pod starts , in such scenario we can use an init container. An initContainer is configured in a pod like all other containers, except that it is specified inside a `initContainers` section, like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ;']
```


## Sidecar 
Side car containers run along with the main application inside the same Pod to provide additional functionality to the main container for example logging, monitoring or security. Side car containers are different from `init` container in regard the life time. Side car container keep running along with the main container (app).  If any of them fails, the POD restarts. They are implemented using the init container with `RestartPolicy` set to always.

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: multi-container-pod
spec:
 containers:
  - name: sidecar
    image: nginx
 initContainers:
  - name: side-car
    image:  my-app
    restartPolicy: Always
```