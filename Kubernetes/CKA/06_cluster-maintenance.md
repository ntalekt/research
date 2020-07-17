# Cluster Maintenance Progress: 9 / 15

## Section 5:108

### OS Upgrades

pod eviction timeout default of 5min.

You can drain the node of all the workloads before doing maintenance `kubectl drain node-1`. Draining a node also marks is as cordoned so it is unschedulable. You need to uncordon the node when it comes back online `kubectl uncordon node-1`.

`kubectl cordon node-2` makes sure no new pods are schedule but doesn't remove the existing pods.

## Section 5:112

### Cluster Upgrade Process

Core components can be at different release versions however non should be higher than kube-apiserver.

-   kube-apiserver = X
-   Controller-manager = X-1
-   kube-scheduler = X-1
-   kubelet = X-2
-   kube-proxy = X-2
-   kubectl X+1 > X-1

Upgrade one minor version at a time.

`kubeadm upgrade plan` give a lot of good information about the possible upgrade. `kubeadm` does not upgrade kubelets.

Upgrade control plane

    apt-get upgrade -y kubeadm=1.18.0-00
    kubeadm upgrade plan
    kubeadm upgrade apply v1.18.0

Upgrade the kubelets

    kubectl drain node01
    apt-get upgrade -y kubelet=1.18.0-00
    systemctl restart kubelet
