## Create different objects quickly

```bash
# Create a Pod
k run nginx --image=nginx

# Create a deployment named my-dep that runs the busybox image and expose port 5701
k create deployment my-dep --image=busybox --port=5701

# Job
k create job my-job --image=nginx --dry-run=client -o yaml

# Cron job
k create cronjob nginx --image=nginx --dry-run=client --schedule="* * * * *" -o yaml

# Config map
k create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2

# Secret
k create secret generic my-secret --from-literal=key1=super-secret --from-literal=key2=top-secret

## TLS Secret
kubectl create secret tls tls-secret --cert=path/to/tls.cert --key=path/to/tls.key

# Service
k create service clusterip my-service --tcp=80:80
```


## Expose different objects quickly

```bash
#Pod
k expose pod nginx --port=80 --target-port=8000

# Deployment
k expose deployment nginx --port=80 --target-port=8000

# Replica set
k expose rs nginx --port=80 --target-port=8000
```

 ## List of resources along with API Version 


```bash
kubectl api-resources
```

## List of API versions
```bash
kubectl api-versions
```

## Get API version of a particular resource
```bash
kubectl explain deployment
```
## Switching a namespace

```bash
kubectl config set-context --current --namespace=<desired-namespace>
```

## Get all resources against a selector

```bash
kubectl get all --selector env=prod
```

## Get metrics of a node/pod

```bash
kubectl top node
kubectl top pod
```



## Helm
```bash
# install a package
helm install release-name wordpress

#upgrade a package
heml upgrade wordpress

#rollback a package
heml rollback wordpress

#uninstall a package
heml uninstall release-name

# search artifact hub
helm search hub wordpress

# for bitnami first add the repo
helm repo add bitnami https://charts.bitnami.com/bitnami

# now search the bitnami repo (notice now we use repo instead of hub)
helm search repo wordpress

# list packages
helm list

# if to only pull the chart
helm pull --untar bitnami/wordpress
```

 # test connectivity of a service from a temporary pod

```bash
kubectl run busybox --rm --image=busybox -it --restart=Never -- wget -O- [PUT THE POD'S IP ADDRESS HERE]:80
```

| Command                 | Meaning                            |
| ----------------------- | ---------------------------------- |
| `wget -O file.html URL` | Save output to **file.html**       |
| `wget -O - URL`         | Send output to **terminal / pipe** |
