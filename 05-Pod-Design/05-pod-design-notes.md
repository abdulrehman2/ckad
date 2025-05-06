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

#Whenever we change the deployment a change-cause is recorded (what has changed), By default the `CHANGE-CAUSE` is not recorded , so we have to explicity pass the --record flag

kubectl rollout history deployment/deployment-name --record
```

## Updating a deployment
```bash
# applying the new template
kubectl apply -f deployment.yaml

# changing the image only
kubectl set image deployment/deployment-name nginx-container=nginx:1.9.1
```

## Rollback /Undo
We can rollback the deployment if something goes wrong, this way the new replica set will be replaced by the older replica set

```bash
kubectl rollout undo deployment/deployment-name
```

# Blue Green Deployment
We deploy a version of app and we route traffic to it, lets call it version-1 this is the `blue`. Later we deploy a newer version (version-2) of the app this is `green`. 
Once everything looks good, the traffic from version-1 is shifted to version-2.


 # Canary Deployment
When we want to deploy two versions of same application, but only want to route small amount of traffic to the new version, we can use the canary deployment.
Lets say we have a deployment with 5 replicas (version-1), we can deploy a new deployment with only one replica and we have a common service that will route traffic between
these two deployments. This way we can achieve canary deployment by using k8s native components.

For this purpose, we have to maintain a certain of replicas for first version and fewer replicas of the second version. Once everything looks good, we delete the first deployment.

```yaml
# First Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
 name: myapp-primary
 labels:
  app: myapp
spec:
 replicas: 5
 selector:
  matchLabels:
   app: frontend
 template:
  metadata:
   labels:
    app: frontend
  spec:
   containers:
    - name: app-container
      image: kodecloud-color:v1
```

```yaml
# Second Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
 name: myapp-canary
 labels:
  app: myapp
spec:
 replicas: 1
 selector:
  matchLabels:
   app: frontend
 template:
  metadata:
   labels:
    app: frontend
  spec:
   containers:
    - name: app-container
      image: kodecloud-color:v2
```

# Jobs
Workloads that are meant to perform a specific task and then stopped should be created as job in kubernetes, some examples of such wokrloads are
- Batch processing
- Analytics
- Generating a report and sending an email

In docker if we try to run a short lived workload, the container will exit after completing the task

```bash
docker run ubuntu expr 3+2
```

If we replicate the same thing in kubernetes we will do something like

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: my-job
spec:
 restartPolicy: Never #By default it is set to `Always` this will cause problem as job will run again and again so we set it to Never
 containers:
  - name: job-container
    image: ubuntu
    command: ['expr','+','3','2']
```

This works if we want a single pod to run the job, what if we want several pods to process a set of datasets ? For this we can use the `Job` object in kubernetes.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
 name: my-job
spec:
 completions: 3   # no of pods that should be created
 parallelism: 3   # by default pods are created in sequence (once one is completed then create next)
 template:
  spec:
   restartPolicy: Never
   containers:
    - name: job-container
      image: ubuntu
      command: ['expr','+','3','2']
```

- Kubernetes will create 3 pods one after the other, if any pods fails, kubernetes will try to create a new one untill the desired state (3 succesfull exits) is acheived.
- By default pods are created in sequential manner and next pod gets created only when previous one is exited, we can change it by using `parallelism` property.


## Cron Jobs
With cron jobs we can schedule to run a job at a particular time and day. the yaml for cron job will look something like this. We will have 3 `specs`. One for cron job, 1 for job template and finally one for container.

```yaml
apiVersion: batch/v1beta1
kind: Cronjob
metadata:
 name: my-cron-job
spec:
 schedule: "*****"
 jobTemplate:
  spec:
   completions: 3
   parallelism: 3
   template:
    spec:
     containers:
      - image: ubuntu
        name: my-pod
        command: ["expr","+","3","2"]
```