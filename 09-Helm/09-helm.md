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