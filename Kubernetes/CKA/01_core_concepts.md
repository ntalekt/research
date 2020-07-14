# Core Concepts Progress: 33 / 44

## Section 2:9

### Control plane components

-   node-controller
-   takes care of nodes, new nodes, unavailable nodes
-   replication-controller
-   makes sure enough containers are running in a specific replication group

-   master node
-   ETCD cluster - key value format
-   kube-scheduler
-   kube-apiserver

    -   orchestrating all operations in a cluster

-   worker node
-   kubelet
    -   runs on each node in the cluster
    -   monitor status of nodes (captain of the ship)
-   kube-proxy
    -   enable communication between worker nodes
-   container runtime (e.g. docker, containerd, rocket)

## Section 2:10

### ETCD for beginners

Port 2379

-   ETCD is a distributed reliable key-value store that is Simple, Secure, & Fast.
-   etcdctl cli client
    Example


    ./etcdctl set key1 value1
    ./etcdctl get key1

## Section 2:11

### ETCD in Kubernetes

-   Stores information related to the cluster
    -   nodes
    -   pods
    -   configs
    -   secrets
    -   accounts
    -   roles
    -   bindings

Multiple master nodes in a cluster, multiple instances of ETCD

## Section 2:13

### Kube-API server

Only component that interacts directly with the ETCD data store

-   Authenticates user
-   Validate request
-   retrieve data
-   update ETCD
-   scheduler
-   kubelet

## Section 2:14

### kube controller manager

-   Intelligence for each construct has been implemented using a controller concept.
-   packaged into a single process (kube-controller-manager)

### View kube-controller-manager - kubeadm

Deployed as pod in the kube-system namespace

    kubectl get pods -n kube-system

View pod definition file

    cat /etc/kubernetes/manifests/kube-controller-manager

### View in Non-kubeadm

manager server service

    cat /etc/systemd/system/kube-controller-manager.service

## Section 2:15

### kube-scheduler

Deciding which pod goes on which nodes

-   looks at the pods and tries to figure out the best node to place
-   Phase1 - filter nodes
-   Phase2 - ranks nodes
-   kubelet creates the pod on the nodes

## Section 2:16

### Kubelet

-   Kindof like the captain of the ship
-   Registers the node with kubernetes cluster
-   Reports back to the cluster

## Section 2:17

### Kube proxy

Every pod can reach every other pod (pod network)

## Section 2:18

### What is a POD?

-   Smallest object in kubernetes
-   Usually a 1:1 mapping to containers
-   Not a strict requirement

### Howto Deploy a POD

    kubectl run nginx --image nginx
    kubectl get pods

## Section 2:19

### POD with YAML

Create new pod definition file

    vi pod-definition.yml

Add required elements

    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
      labels:
          app: myapp
          type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx

Create pod

    kubectl create -f pod-definition.yml

View all pods

    kubectl get pods

View specifics about a pod

    kubectl describe pod myapp-pod

## Section 2:24

### Practice Test - Pods

100%

## Section 2:25

### Replica

Replica - For HA more then one pod should be running for an application. Replication controller helps run multiple pods per application instance.

#### Replication Controller

-   Replication Controller - Older technology
-   Replication of pods for HA
-   Load balancing
-   Scaling

rc-definition.yml

    apiVersion: v1
    kind: ReplicationController
    metadata:
      name: myapp-rc
      labels:
          app: myapp
          type: front-end
    spec:
      template:
        metadata:
          name: myapp-pod
          labels:
              app: myapp
              type: front-end
        spec:
          containers:
            - name: nginx-container
              image: nginx
      replicas: 3

Create the Pods

    kubectl create -f rc-definition.yml

View replication controller

    kubectl get replicationcontroller

View pods that were created

    kubectl get pods

#### ReplicaSets

-   Replica Set - Newer technology
-   Uses a different api
-   Has selector input
    replicaset-definition.yml


    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: myapp-replicaset
      labels:
          app: myapp
          type: front-end
    spec:
      template:
        metadata:
          name: myapp-pod
          labels:
              app: myapp
              type: front-end
        spec:
          containers:
            - name: nginx-container
              image: nginx
      replicas: 3
      selector:
          matchLabels:
              type: front-end

Create the pods

    kubectl create -f replicaset-definition.yml

View replica sets

    kubectl get replicaset

View pods that were created

    kubectl get pods

##### ReplicaSets Update parameters

To apply a deploy config

    kubectl replace -f replicaset-definition.yml

Edit running

    kubectl edit rs ReplicaSet_name

Scale running

    kubectl scale --replicas=5 rs new-replica-set

## Section 2:29

### Deployments

deployment-definition.yml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: myapp-deployment
      labels:
          app: myapp
          type: front-end
    spec:
      template:
        metadata:
          name: myapp-pod
          labels:
              app: myapp
              type: front-end
        spec:
          containers:
            - name: nginx-container
              image: nginx
      replicas: 3
      selector:
          matchLabels:
              type: front-end

Deploy

    kubectl create -f deployment-definition.yml

View

    kubectl get deployments

View ReplicaSet

    kubectl get replicaset

View Pods

    kubectl get pods

Get all objects

    kubectl get all

#### Tips

Create an NGINX Pod

    kubectl run --generator=run-pod/v1 nginx --image=nginx

Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

    kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml

Create a deployment

    kubectl create deployment --image=nginx nginx

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

    kubectl create deployment --image=nginx nginx --dry-run -o yaml

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)

    kubectl create deployment --image=nginx nginx --dry-run -o yaml > nginx-deployment.yaml
