# Troubleshooting Progress: 12 / 12

## Section 13:226

### Application Failure

-   Service that exposed mysql via ClusterIP was named `mysql` instead of `mysql-service`
-   mysql-service had an incorrect TargetPort

## Section 13:229

### Control Plane Failure

    kubectl get nodes
    kubectl get pods

Control plane deployed as pods

    kubectl get pods -n kube-system

Control plane deployed as services

    service kube-apiserver status
    service kube-controller status
    service kube-scheduler status

worker nodes

    service kubelet status
    service kube-proxy status

Deployed via kubeadm

    kubectl logs kube-apiserver-master -n kube-system

Deployed via services

    sudo journalctl -u kube-apiserver

## Section 13:232

### Worker Node Failure

    kubectl get nodes
    kubectl describe node worker-1

On nodes

    top
    df -h
    service kubelet status
    sudo journalctl -u kubelet
    openssl x509 -in /var/lib/kubelet/worker-1.crt -text
