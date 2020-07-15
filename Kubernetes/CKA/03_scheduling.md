# Scheduling Progress: 10 / 29

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
