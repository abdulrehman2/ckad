1. ## List of resources along with API Version 
```bash
kubectl api-resources
```

2. ## List of API versions
```bash
kubectl api-versions
```

3. ## Get API version of a particular resource
```bash
kubectl explain deployment
```
4. Switching a namespace

```bash
kubectl config set-context --current --namespace=<desired-namespace>
```

5. Get all resources against a selector

```bash
kubectl get all --selector env=prod
```

6. Get metrics of a node/pod

```bash
kubectl top node
kubectl top pod
```


# 9. Helm
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

