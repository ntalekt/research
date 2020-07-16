# Scheduling Progress: 29 / 29

## Section 3:46

### Manual Scheduling

Schedule the pods yourself by using the `nodeName` parameter under spec.

    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
      labels:
          name: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
          - containerPort 8080
      nodeName: node02

## Section 3:50

### Labels & Selectors

Standard method of grouping things together

    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
      labels:
          name: nginx
          app: App1
          function: Front-end
      annotations:
          buildversion: 1.34
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
          - containerPort 8080
      nodeName: node02

Select pods using selector

    kubectl get pods --selector app=App1

All objects with labels

    kubectl get all --selector env=prod

Select pods with multiple labels

    kubectl get pods -l env=prod,bu=finance,tier=frontend

## Section 3:53

### Taints and Tolerations

Set restriction to what pods are scheduled on specific nodes.
Taints are set on nodes and tolerations are set on pods
Taint a node

    kubectl taint nodes node01 spray=mortein:NoSchedule

-   **NoSchedule** - don't schedule pods that can't tolerate the taint
-   **PreferNoSchedule** - Prefer to not schedule but schedule if nothing else
-   **NoExecute** - don't schedule pods that can't tolerate the taint and evict already scheduled pods


    apiVersion: v1
    kind: Pod
    metadata:
      name: bee
    spec:
      containers:
      - name: bee
        image: nginx
      tolerations:
      - key:"spray"
        operator: "Equal"
        value:"mortein"
        effect:"NoSchedule"

Find taints on a host

        kubectl describe node node01 | grep Taint

Remove a taint

        kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-

## Section 3:55

### Node selectors

Can use the pod definition to sepcific a nodeSelector

    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
    spec:
      containers:
      - name: bee
        image: nginx
      nodeSelector:
        size: Large

Label a node

    kubectl label nodes node01 size=Large

## Section 3:56

### Node affinity

To ensure pods are scheduled on specific nodes. Enables advanced expressions.

    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
    spec:
      containers:
      - name: bee
        image: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: size
                operator: In
                values:
                - Large
                - Medium

#### Node Affinity Types

Available

-   requiredDuringSchedulingIgnoredDusringExecution
-   preferredDuringSchedulingIgnoredDuringExecution

Planned

-   requiredDuringSchedulingRequiredDuringExecution

Label nodes

    kubectl label nodes node01 color=blue

## Section 3:58

### Taints and Tolerations vs Node Affinity

Can b e used to fully dedicate hardware. Taint the nodes (red, green, blue) then add the toleration to the pods. Then add affinity to make sure they are selected to be scheduled on the correct nodes.

## Section 3:59

### Resource Requirements and Limits

default is set at the namespace level.

-   <https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/>
-   <https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/>

Extract running pod definition

    kubectl get pod elephant -o yaml > elephant.yaml

## Section 3:64

### DaemonSets

One copy of pod exists on each node in the cluster (e.g. monitoring solution or log viewer).

    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: monitoring-daemon
    spec:
      template:
        metadata:
          labels:
              app: monitoring-agent
        spec:
          containers:
            - name: nginx-container
              image: nginx
      selector:
          matchLabels:
              app: monitoring-agent

View DaemonSets

    kubectl get daemonsets

## Section 3:67

### Static pods

Node/kubelet not part of a cluster.

Can place pod definition files in `/etc/kubernetes/manifests`. Kubelet periodically checks the folder

Check kubelet.service file for `--pod-manifest-path` if it's not there check for `--config`.

Use `docker ps` to view the pods.

## Section 3:69

### Multiple Schedulers

You can instruct the pod descriptor file to use a specific scheduler. `default-scheduler`

-   \--scheduler-name=my-custom-scheduler

Use pod definition file

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
      schedulerName: my-custom-scheduler

View logs of the custom Schedulers

    kubectl logs my-custom-scheduler --name-space=kube-systemd

    netstat -natulp | grep 10259

## Section 3:72

### Configuring Kubernetes Scheduler
