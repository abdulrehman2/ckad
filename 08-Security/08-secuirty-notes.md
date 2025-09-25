# Authentication
Kubernetes does not have an object called user, nor does it store usernames or other related details in its object store. However, even without that, Kubernetes can use usernames for the Authentication phase of the API access control, and to request logging as well.

## Normal Users
They are managed outside of the Kubernetes cluster via independent services like User/Client Certificates, a file listing usernames/passwords, Google accounts, etc.

## Service Accounts
Service Accounts allow in-cluster processes to communicate with the API server to perform various operations. Most of the Service Accounts are created automatically via the API server, but they can also be created manually. The Service Accounts are tied to a particular Namespace and mount the respective credentials to communicate with the API server as Secrets.

In order to limit access to cluster for admins and devs, we can apply authentication mechanism using following techiniques supported.

- Static file based username/password
- Static token file
- Certifcate Based
- LDAP or external Identity provider

## Basic (file based username/password)

A csv file should have 3 mandatory columns and 1 optional column `usergroup`

password,username,userId

we pass the user file to kube-apiserver.service as an argument to setup the basic authentication mechanism

```bash
ExecStart= /usr/local/bin/kube-apiserver \\
--basic-auth-file=user-details.csv
```
We want to authenticate the user, we can do it using the 

```bash
curl -v -k https://master-node-ip:6443/api/v1/pods - u "user1:password123"
```
## Static Token File
Instead of storing password, we can have tokens

in csv file we will then have

`longEncryptedToken`,`username`,`userId`,`userGroup`

```bash
ExecStart= /usr/local/bin/kube-apiserver \\
--token-auth-file=user-details.csv
```

We want to authenticate the user, we can do it using the 

```bash
curl -v -k https://master-node-ip:6443/api/v1/pods  --header "Authorization: Bearer <longEncryptedToken>"
```

> Note these mechanisms are not recommended, since we are storing the users/password in static files.Also note that this approach is deprecated in Kubernetes version 1.19 and is no longer available in later releases.Instead consider using a volume mount while providing the auth file in a kubeadm setup

# API Groups
All resources in kubernetes are grouped into API groups. In order to communicate with K8s, we interact via api-server using either `kubectl` or through REST API for example 

```bash
curl https://kube-master:6443/version
curl https://kube-master:6443/api/v1/pods
```

k8s api's are grouped into multiple groups based on their purpose for example

- /version  # getting version of cluster
- /api      # cluster functionality (Core group)=>      namespace, events, configmaps, nodes, pods, services, secrets, endpoints, events, rc, bindings pv, pvc etc. 
- /apis     # cluster functionality (Named group) =>    
- /healthz
- /metrics
- /logs  # integrating logs with 3rd party tools

 The named group has following API groups
- /apps
  - /v1
    - /deployments (list, get, create,delete, update, watch)
    - /replicaset
    - /statefulset
- /extensions
- /networking.k8s.io
  - /v1
    - /networkpolicies
- /storage.k8s.io
- /authentication.k8s.io
- /certificates.k8s.io
  - /v1
    - /certificatesigningrequests

```bash
curl https://kube-master:6443 -k # this will display all the group paths for example /api, /api/v1, /apis, /healthz etc.
curl https://kube-master:6443 -k | grep "name" # this will display resource groups withing the group
```

 if we try to access the cluster using `curl`, we will be limited to few operations only like getting the version. If we want to perform more advance operations we have to get authenticated using the certificates.

```bash
curl http://localhost:6443 -k --key admin.key --cert admint.crt --cacert ca.crt
```

Another option is to setup a **kubectl proxy** 

```bash
kubectl proxy # this launches a proxy service locally on port 8001 and uses the certificate from kube config file to access the cluster. This way we don't need to pass the certificate details in the curl command
```
Proxy will forware the user request to the API server.

user => Kubectl proxy => Kube ApiServer

| Feature           | `kubectl proxy`                       | `kube-proxy`                               |
| ----------------- | ------------------------------------- | ------------------------------------------ |
| **Where it runs** | On your local machine                 | On every Kubernetes node                   |
| **Purpose**       | Access Kubernetes API securely        | Enable Services & Pod-to-Pod communication |
| **Scope**         | API requests only                     | All cluster Service networking             |
| **Lifecycle**     | Manual, temporary (until you stop it) | Automatic, always running                  |
| **Type**          | CLI developer tool                    | Cluster networking component               |


Also note the that **kube proxy** != **kubectl proxy**



# Authorization

Once a user is authenticated, the next step in the process is to make sure what actions a user can perform. An admin can perform all the operations on the cluster, but we don't want every user to have same kind of rights. We partition our cluster using namespaces for different teams and organizations, we can restrict the users to their relevant namespaces alone.

## Node authorization
User request `kube-api` server to perform certain operations, similarly `kubelet` (it is an agent that run on every worker node) report certain details of the node to the Kube-Api server. This action is allowed by a special authorizer, known as `Node Authorizer`

 **What the Node Authorizer does**

- When enabled (it is by default), the Node Authorizer enforces rules so that a kubelet can only:
- Read/modify its own Node object (not others).
- Read/watch Pods bound to itself.
- Read Secrets, ConfigMaps, PVCs, and ServiceAccounts only if those are used by its Pods.
- Post Node status updates for itself.

It pairs with the `NodeRestriction` admission plugin, which further restricts what kubelets can modify.

## ABAC (Attribute-Based Access Control)
A user or group of users are allowed to perform certain opertions within the cluster. For example dev users can create, read, delete pods. For that we create a policy in JSON format and pass this file to api server.
- If we want to make any changes to secuirty of these users, we have modify it for each user and restart the api server (not very convinient)
```json
{
    "kind":"Policy",
    "spec":{
        "user": "dev-user",
        "namespace": "*",
        "resource":"pods",
        "apiGroup": "*"
    }
},
{
    "kind":"Policy",
    "spec":{
        "user": "dev-user-2",
        "namespace": "*",
        "resource":"pods",
        "apiGroup": "*"
    }
},
{
    "kind":"Policy",
    "spec":{
        "group": "dev-users",   // group
        "namespace": "*",
        "resource":"pods",
        "apiGroup": "*"
    }
},
```

## Role based authorization (RBAC)
Instead of managing the user rights at the user level, 
- we create a standard role for developers, security users etc
- assign permission to these roles and then we assign this role to the user.
- this way we only have to change in one place later.

First create a role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
 name: developer
rules:
 - apiGroups: [""]
   resources: ["pods"]
   verbs: ["list","create","update","delete", "watch"]
   resourceNames: ["blue", "green"] # if we want to give access to particular pods only
 - apiGroups: [""]
   resources: ["ConfigMap"]
   verbs: ["create"]
```

Then create a role binding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
 name: devuser-developer-binding
subjects:
 - kind: User
   name: dev-user
   apiGroup: rbac.authorization.k8s.io
roleRef:
 kind: Role
 name: developer # created above
 apiGroup: rbac.authorization.k8s.io
```

How to check if i have access to perform a particular action

```bash
kubectl auth can-i create deployments
kubectl auth can-i create delete-nodes

# if we want to test for a particular user
kubectl auth can-i create delete-nodes --as dev-user 

# if we want to test for a particular user in a particular namespace

kubectl auth can-i create delete-nodes --as dev-user  --namespace test
```


## Webhook
If we dont want to manage the access of users inside the kubernetes, and let it manage somewhere else, we can use `Open Policy Agent`
- k8s will send a request to the agent to verify if user is allowed to perform the action
- agent will allow or deny the request

## AlwaysAllow
Allow all request, this can be configured in kube api server, we have the flag `--authorization-mode`. Be default it is set to `AlwaysAllow`

## AlwaysDeny
Deny all request, this can be configured in kube api server, we have the flag `--authorization-mode`

We can also set multiple modes --authorization-mode=Node, RBAC, Webhook. The authorization will be check in this order

Node => RBAC => Webhook

> If permission is denied at any module, the request will be forwarded to next mode. Once a request is approved by any module, the request will be completed.

