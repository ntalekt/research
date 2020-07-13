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
