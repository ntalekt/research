# Scheduling Progress: 8 / 29

## Section 3:46

### Manual Scheduling
Schedule the pods yourself by using the `nodeName` parameter under spec.

```
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
```

## Section 3:50

### Labels & Selectors
Standard method of grouping things together
```
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
```

Select pods using selector
```
kubectl get pods --selector app=App1
```
## Section 3:53

### Taints and Tolerations
