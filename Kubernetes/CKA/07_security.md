# Security Progress: 23 / 29

## Section 6:123

### Kubernetes Security Primitives

-   Secure hosts via SSH key based auth.

-   Controlling access to the kube-apiserver is the first line of security.

## Section 6:124

### Authentication

All user access is managed by the kube-apiserver.

#### Auth Mechanisms - Basic

Not a recommended approach for authentication
user-details.csv

    password123,user1,u0001,group1
    password123,user2,u0002,group2

## Section 6:128

### TLS Basics

#### PKI - Public Key Infrastructure

-   Symmetric Encryption
-   Asymmetric Encryption - uses a pair of keys (private and public)

1.  User browses to website
2.  Website sends user public key
3.  User encrypts symmetric key with public key
4.  Server uses private key to descript and receives symmetric key
5.  User and server now send encrypted messages using symmetric key

## Section 6:129

### TLS in Kubernetes

#### Server certs for Servers

-   kube-api
-   etcd server
-   kubelet

#### Client certs

-   admin
-   kube-scheduler
-   kube-controller-manager
-   kube-proxy

## Section 6:130

### TLS in Kubernetes - Certificate Creation

#### CA Certificate

Generate key -
`openssl genrsa -out ca.key 2048`

CSR -
`openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr`

Sign cert -
`openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt`

#### Client Certificate

Generate key -
`openssl genrsa -out admin.key 2048`

CSR -
`openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr`

Sign cert with CA
`openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt`

## Section 6:131

### View Certificate details

Find `kube-apiserver.yaml` under `/etc/kubernetes/manifests`

#### View details

`openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout`

`kubectl logs etcd-master`

## Section 6:134

### Certificates API

1.  User first creates a key `openssl genrsa -out jane.key 2048`
2.  User creates CSR using the key `openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr`

Administrator creates a certificate signing object

    apiVersion: certificates.k8s.io/v1beta1
    kind: CertificateSigningRequest
    metadata:
      name: jane
    spec:
      groups:
        - system:authenticated
      usages:
        - digital signature
        - key encipherment
        - server auth
      request:

Encode the CSR using base64 and put into request section `cat jane.csr | base64`

View CSRs

    kubectl get csr

Approve CSR

    kubectl certificate approve jane

View the certificate

    kubectl get csr jane -o yaml

Decode the cert

    echo "LS0....q" | base64 --Decode

View the cluster signing cert as part of hte controller-manager configuring `cat /etc/kubernetes/manifests/kube-controller-manager.yaml`

## Section 6:136

### KubeConfig

Default location is `$HOME/.kube/config`

-   Clusters
-   Contexts
-   Users

View the current config

    kubectl config view

View a different config

    kubectl config view --kubeconfig=my-custom-config

Change Contexts

    kubectl config use-context prod-user@production

## Section 6:139

### API Groups

Focusing on APIs responsible for the clusters functionality.

-   Core group `/api` where core functionality like namespaces, pods, events, nodes etc.
-   named group `/apis` are more organized like `/apps`, `/extensions`, `/networking.k8s.io` etc

## Section 6:140

### RBAC

Create a role

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: developer
    rules:
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["list", "create"]

Bind a role to a user

    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: dev-user-binding
    subjects:
    - kind: User
      name: dev-user
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: Role
      name: developer
      apiGroup: rbac.authorization.k8s.io

-   Get roles `kubectl get roles`
-   Get role bindings `kubectl get rolebindings`
-   Get more details about a role `kubectl describe role developer`
-   Get more details about a role binding `kubectl describe rolebinding devuser-developer-binding`

#### Check access

    kubectl auth can-i create deployments
    kubectl auth can-i delete nodes
    kubectl auth can-i create deployments --as dev-user
    kubectl auth can-i create pods --as dev-user

## Section 6:142

### Cluster Roles and Role bindings

Create a cluster role

    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: cluster-administrator
    rules:
    - apiGroups: [""]
      resources: ["nodes"]
      verbs: ["list", "get", "create", "delete"]

Create a cluster role binding

    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: cluster-role-binding-michelle
    subjects:
    - kind: User
      name: michelle
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: ClusterRole
      name: cluster-administrator
      apiGroup: rbac.authorization.k8s.io
