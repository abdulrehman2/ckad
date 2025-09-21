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


> Note these mechanisms are not recommended, since we are storing the users/password in static files.Also note that this approach is deprecated in Kubernetes version 1.19 and is no longer available in later releases.Instead consider using a volume mount while providing the auth file in a kubeadm setup

# API Groups


