---
title: "Installing MongoDB in Kubernetes"
date: 2023-02-26
tags: ["DevOps","MongoDB","Operator"]
authors: ["kevinan9"]
---

# Installing MongoDB in Kubernetes

## Prerequisites

This tutorial will be a hands-on demonstration. To follow along, be sure you have the following requirements:

-   `minikube` up and running - check out our setup guide: [[WIP] Local Kubernetes Setup](https://lightspeedteamr.atlassian.net/l/cp/HtmQ781f "https://lightspeedteamr.atlassian.net/l/cp/HtmQ781f")
    
-   [MongoDB Compass](https://www.mongodb.com/products/compass "https://www.mongodb.com/products/compass") installed on the local machine.
    
-   [Git](https://git-scm.com/downloads "https://git-scm.com/downloads") installed on your machine.
    
-   [jq](https://stedolan.github.io/jq/ "https://stedolan.github.io/jq/") JSON Parser installed on the local machine.
    

## Downloading the MongoDB Kubernetes Operator

Run the `git` command below to `clone` the MongoDB Kubernetes Operator project to your local machine.

```git clone https://github.com/mongodb/mongodb-kubernetes-operator.git```

Now, run each command below to change your current working directory to `mongodb-kubernetes-operator` and list (`ls`) all available files and directories.

```
# Change the working directory to mongodb-kubernetes-operator  
cd mongodb-kubernetes-operator/ 

# List files and directories
ls
```

## Deploying the MongoDB Operator

Run below commands

```
kubectl create ns mongodb 
 
# create a new Kubernetes CRD for MongoDB deployment. 4
kubectl apply -f config/crd/bases/mongodbcommunity.mongodb.com_mongodbcommunity.yaml 

# create a new custom Role-Based Access Control (RBAC) for the MongoDB Operator, and specify RBAC implementation to the namespace mongodb. 

kubectl apply -k config/rbac/ -n mongodb 

# This command creates a new pod (mongodb-kubernetes-operator) with the base Docker image (quay.io/mongodb/mongodb-kubernetes-operator). This pod will act as the controller for automatically deploying MongoDB ReplicaSets on the Kubernetes cluster.
kubectl create -f config/manager/manager.yaml -n mongodb
```

## Deploying MongoDB ReplicaSet to Kubernetes

**Firstly**, replace <your-password-here> with your strong password in the file `config/samples/arbitrary_statefulset_configuration/mongodb.com_v1_hostpath.yaml`.

```
... 
users: 
- name: mongoadmin 
db: admin 
passwordSecretRef: # a reference to the secret that will be used to generate the user's password 
name: my-user-password 
---
 apiVersion: v1 
 kind: Secret 
 metadata: 
  name: my-user-password 
  type: Opaque 
  stringData: 
  password: <your-password-here> # Set password for MongoDB admin
```

**Next**, run the `kubectl` command below to deploy (`apply`) the MongoDB ReplicaSet, which will take some time.

```
kubectl apply -f config/samples/arbitrary_statefulset_configuration/mongodb.com_v1_hostpath.yaml -n mongodb 

# creates a new port forwarding on the Kubernetes service (mdb0-svc) and forwards the local port 27017 to the Kubernetes service port 27017. 
kubectl port-forward service/mdb0-svc -n mongodb 27017:27017
```

**Finally**, open your MongoDB Compass application on your local machine, add a new connection with the following format, and click **Connect** to connect to MongoDB.

`mongodb://mongoadmin:your-password-here@localhost:27017/admin?ssl=false`

![](https://user-images.githubusercontent.com/1268001/221662725-5f6a3d39-5f67-4490-8f0f-d440dd596a72.png)

- The End -