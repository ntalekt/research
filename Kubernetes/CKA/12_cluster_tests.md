# End to End Tests on a Kubernetes Cluster Progress: 4 / 4

## Section 12:220

### End to End Tests

#### Test Manual

-   kubectl get nodes
-   kubectl get pods
-   kubectl get pods -n kube-system #control plane as pods
-   service kube-apiserver status #control plane as service
-   service kube-controller-manager status #control plane as service
-   service kube-schedular status #control plane as service
-   service kubelet status #control plane as service
-   service kube-proxy status #control plane as service
-   kubectl run nginx
-   kubectl get pods
-   kubectl scale --replicas=3 deploy/nginx
-   kubectl get pods
-   kubectl expose deployment nginx --port=80 --type=NodePort
-   kubectl get service

#### Kubetest

-   <https://github.com/kubernetes/test-infra/tree/master/kubetest>
-   <https://github.com/kubernetes-sigs/kubetest2>

#### Sonobuoy

Easier to setup and configure.

## Section 12:221

### End to End Tests - Run and Analyze

-   Download the tests `go get -u k8s.io/test-infra/kubetest`
-   Get the binaries needed for the test `kubetest --extract=v1.11.3` # Note: Version must match the kubernetes server version
-   After extracted `cd kubernetes`
-   Set the KUBE_MASTER_IP and KUBE_MASTER (`export KUBE_MASTER_IP="192.168.26.10:6443"` and `export KUBE_MASTER=kube-master`)
-   Then launch the test with `kubetest --test --provider=skeleton > testout.txt`
