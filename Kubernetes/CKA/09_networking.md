# Networking Progress: 20 / 27

## Section 9:168

### Network Namespaces

Used by containers to implement network isolation.

Create new network namespace `ip netns add red`
View network namespace `ip netns`
Run command inside namespace `ip netns exec red ip link`

## Section 9:175

### CNI in Kubernetes

Look at the kublet service for CNI information `ps -aux | grep kubelet`

    --network-plugin=cni
    --cni-bin-dir=/opt/cni/bin

View the CNI plugin that is configured `ls /etc/cni/net.d/`

## Section 9:177

### CNI weave

Works by installing agents on each node. They communicate with each other and stores a topology of the cluster.

View routes on container `kubectl exec busybox ip route`

Deploy weave as pods in the cluster `kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"`

## Section 9:179

### IP Address Management - Weave

-   CNI plugin to manage IP assignments.
-   `cat /etc/cni/net.d/net-script.conf` shows the subnets and routes to be used.

## Section 9:181

### Service Networking

-   ClusterIP - When a service is created all pods on all nodes in the cluster can access it
-   NodePort - When a service is created all pods on all nodes in the cluster can access it and exposes the service via an external port.

View IP range configured for services

    cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep cluster-ip-range
