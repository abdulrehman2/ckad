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
kubectl delete -k k8s/
```

## Managing Directories
Previoulsy we only had a single directory that hosts our yaml files, and we can apply all the changes by using

`kubectl apply -f k8s/`
but what if we have structure like this
- k8s
  - api
    - api-depl.yaml
    - api-service.yaml
  - db
    - db-depl.yaml
    - db-service.yaml
  - cache
    - redis-depl.yaml
    - redis-service.yaml
  - kafka
    - kafka-depl.yaml
    - kafka-service.yaml
then we have to go to each directory individually and apply one by one like

`kubectl apply -f k8s/api`
then
`kubectl apply -f k8s/db`
....

This is not very scalable since the number of files can grow. 
- We can fix that by adding a `kustomization.yaml` file at the root directory i.e. `k8s`
- Also add `kustomization.yaml` to the sub directories, where subdirectory kustomization file will list resources for that particular directory.
- And in the root file we can mention what directories we want to include in build.

### Subdirectory kustomization file example
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
 - api-depl.yaml
 - api-service.yaml
```

### Root k8s file
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:                  # we just add the paths of sub directories
 - api/
 - db/
 - cache/
 - kafka/
```

## Transformers
Transformers allow us to transform the k8s manifest files, using different transformers. Lets say 
- we want to prefix every object in k8s with dev or prod, we can do that by using  `namePrefix/Suffix` transformer
- we want to add common labels in all objects using  `commonLabel` transformer
- we can add namespace to all resource using `Namespace` transformer
- we can add `commonAnnotations` to all resources

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
 - api/api-depl.yaml
 - api/api-service.yaml
commonLabels:
 org: CompanyX    # add this label to all resources
namespace: lab    # use this namespace for all resources
namePrefix: apple # add the prefix to name of all resources
nameSuffix: -dev  # add the suffix to name of all resources
commonAnnotations:
 branch: master
```

### Image Transformers
We can also transform the image for the containers
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
 - api/api-depl.yaml
 - api/api-service.yaml
images:
 - name: nginx
   newName: haproxy     # replace any pod that has nginx image with haproxy
   newTag: "2.4"        # we can also only use the newTag to change the tag for nginx
```

## Patches
Another method to modify kubernetes configs. Patches provides a more surgical approach to target one or more sections in a kubernetes resource.

Every patch operation has 3 values

1. Operation Type: add/remove/replace
2. Target (what resource this patch will be applied on)
 - Kind
 - Version/Group
 - Name
 - Namespace
 - labelSelector
 - AnnotationSelector
3. Value (value with which the property will be replaced or added)

### JSON Patch
Lets say we have a deployment with name `api-deployment` and want to replace it with `web-deployment`.

Using the JSON 6902 Patch specification we can have
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
patches:
 - target:
    kind: Deployment
    name: api-deployment
   patch: |-
    - op: replace
      path: /metadata/name
      value: web-deployment
```
We can also move the patches to a seperate file to avoid clutering.

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
patches:
 - patch: replica-patch.yaml
   target:
    kind: Deployment
    name: api-deployment
```

and replicate-patch.yaml will look like
```yaml
- op: replace
  path: /spec/replicas
  value: 5
```


### Strategic Patch
On other hand we also have strategic merge patch. Strategic patch looks like more a k8s resource file.

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
patches:
   - patch: |-
     apiVersion: apps/v1
     kind: Deployment     
     metadata:
      name: api-deployment    # what resource we want to update
     spec:
      replicas: 5             # anything that does not match with original, will be replaced with this
```

## Overlays
At the begining of section we saw something like a folder structure like this
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

Base structure will have all the files that are common and needs to be deployed. Overlay folder will contain the patches and kustomization file for each environment we are targeting.

Base/kustomization will look like

```yaml
resources:
 - nginx-depl.yaml
 - service.yaml
 - redis-depl.yaml
```

While the dev/kustomization may look like this
```yaml
bases:
 - ../../base   # path to the base folder

patch: |-
 - op: replace
   path: /spec/replicas
   value: 3
```


## Components
Components provide a way to support multiple optional resources/features that needs to be enabled only in a subset of overlays