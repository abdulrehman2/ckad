Kustomize is built in tool into kubectl, that help us to customize the k8s manifest files according to the environments. In kustomize  we have base configs and overlays.
- Kustomize is much simpler as compared to hem , which has complex templating
- plain yaml files 
- dont need to install like helm as it comes with kubectl.
- Helm has lot more features like conditions, loops, functions, hooks etc.
- Helm also act as a package manager for different charts 

# Base config
Any K8s resources which will be same across all different environments are classified as base config. You will also provide default values for any configurations that may be different based on the environment.

# Overalys 
This will hold any diversions from the base config. For example the number of replica for a deployment, we can set different replica value for different environments.
 
 - k8s
   - base
     - kustomization.yaml
     - nginx-depl.yaml
     - service.yaml
     - redis-depl.yaml
   - overlays
     - dev
        - kustomization.yaml
        - config-map.yaml
     - stg
        - kustomization.yaml
        - config-map.yaml
     - prod
        - kustomization.yaml
        - config-map.yaml

`kustmization.yaml` has following contents

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:                  # list all the resources that kustomize should use.
 - nginx-deployment.yaml
 - nginx-service.yaml
commonLabels:               # apply label to all the reosources listed above.
 company: X
```

```bash
kustomize build k8s/                            # apply the customization and output the result in terminal

kustomize build k8s/ | kubectl apply -f -       # apply the customization and deploy the resource in cluster
# or
kubectl apply -k k8s/                           # the k flag is for kustomize

kustomize build k8s/ | kubectl delete -f -      # delete the resource
#or
kubectl delete -f k8s/

```