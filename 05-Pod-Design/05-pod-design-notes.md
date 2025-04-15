# Labels

- Labels are added to kubernetes object to identify a particular set of objects
- Group them based on certain criteria so that later they can be filtered out.

# Selectors
- Selector allow to filter the k8s objects based on the labels
- 

Filter pods based on a set of labels
```bash
kubectl get pods --selector app=web,tier=frontend
```

For replica sets
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
 labels:
  app: backend-replica-set
spec:
 replicas: 2
 selector:
  matchLabels:
    app: backend-app 
 template:
  metadata:
   labels:
    app: backend-app
  spec:
   containers:
    - name: backend
      image: backend-image:0.9
```

Similarly in services we define the selector like this

```yaml

apiVersion: v1
kind: Service
metadata:
 app: backend-service
spec:
 selector:
  app: backend-app
ports:
 - protocol:
   port: 80
   targetPort: 8080 
```

# Annotations
- Annotations are used to record other details of the object, like the version, author name etc.

```yaml
apiVersion: v1
kind: Service
metadata:
 app: backend-service
 annotations: 
  custom.annotation: hello
spec:
 selector:
  app: backend-app
```

# Deployment Strategy

When we update a deployment for example change the labels, update the docker image version etc, then the kubernetes will create a rollout for the changes.

- `Recreate` strategy delete the old pods all at once and recreate the new pods, this will result in downtime as all the pods will be first deleted and then recreated.

- `RollingUpdate` this strategy is more application friendly as it will only down one pod at a time and replace it with newer version and then go for the next pod, one by one (this is the default strategy) 


> Under the hood, when we update a replica set, then the k8s will create a new replicate set, terminating each pod one at a time and replacing it with new 
pod in the new set.

## Rollout Status
To check the rollout of a deployment we can use following command
```bash
kubectl rollout status deployment/deployment-name
```

## Rollout History
```bash
kubectl rollout history deployment/deployment-name
```

## Updating a deployment
```bash
# applying the new template
kubectl apply -f deployment.yaml

# changing the image only
kubectl set image deployment/deployment-name nginx-container=nginx:1.9.1
```

## Rollback 
We can rollback the rollout if something goes wrong, this way the new replica set will be replaced by the older replica set

```bash
kubectl rollout undo deployment/deployment-name
```