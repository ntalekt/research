## Section 2:9
Control plane components
* node-controller
 * takes care of nodes, new nodes, unavailable nodes
* replication-controller
 * makes sure enough containers are running in a specific replication group

* master node
 * ETCD cluster - key value format
 * kube-scheduler
 * kube-apiserver
   * orchestrating all operations in a cluster

* worker node
 * kubelet
   * runs on each node in the cluster
   * monitor status of nodes (captain of the ship)
 * kube-proxy
   * enable communication between worker nodes
 * container runtime (e.g. docker, containerd, rocket)

## Section 2:10
ETCD for beginners
Port 2379
* ETCD is a distributed reliable key-value store that is Simple, Secure, & Fast.
* etcdctl cli client
Example
```
./etcdctl set key1 value1
./etcdctl get key1
```

## Section 2:11
ETCD in Kubernetes
* Stores information related to the cluster
  * nodes
  * pods
  * configs
  * secrets
  * accounts
  * roles
  * bindings

Multiple master nodes in a cluster, multiple instances of ETCD

## Section 2:13
Kube-API server
Only component that interacts directly with the ETCD data store
* Authenticates user
* Validate request
* retrieve data
* update ETCD
* scheduler
* kubelet

## Section 2:14
kube controller manager
* Intelligence for each construct has been implemented using a controller concept.
* packaged into a single process (kube-controller-manager)

### View kube-controller-manager - kubeadm
Deployed as pod in the kube-system namespace
```
kubectl get pods -n kube-system
```
View pod definition file
```
cat /etc/kubernetes/manifests/kube-controller-manager
```

### View in Non-kubeadm
manager server service
```
cat /etc/systemd/system/kube-controller-manager.service
```
