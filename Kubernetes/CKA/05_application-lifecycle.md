# Application Lifecycle Management Progress: 12 / 23

## Section 4:85

### Rolling Updates and Rollbacks

Show the status of the rollout

    kubectl rollout status deployment/myapp-deployment

View rollout history

    kubectl rollout history deployment/myapp-deployment

#### Deployment strategy

-   Recreate - destroy all and redeploy with new version
-   Rolling Update (default) - destroy and redeploy one pod at a time

New rollout is kicked off once the `kubectl apply -f deployment-definition.yaml` command is ran against an updated deployment definition file.

Another way is the use `kubectl set image deployment/myapp-deployment nginx=nginx:1.91`

Possible to roll back the update

    kubectl rollout undo deployment/myapp-deployment

Creates a deployment and replicaset behind the covers.

    kubectl run nginx --image=nginx

## Section 4:90

### Commands and Arguments

Command in Kubernetes definition file overwrites the entrypoint in docker. The args overwrites the CMD in docker.

    apiVersion: v1
    kind: Pod
    metadata:
      name: ubuntu-sleeper-pod
    spec:
      containers:
        - name: ubuntu-sleeper
          image: ubuntu-sleeper
          command: ["sleep2.0"]
          args: ["10"]

## Section 4:92

### Configure Environment Variables in Applications

#### Plain Key Value

    apiVersion: v1
    kind: Pod
    metadata:
      name: simple-webapp-color
    spec:
      containers:
        - name: simple-webapp-color
          image: simple-webapp-color
          ports:
            - containerPort: 8080
          env:
            - name: APP_COLOR
              value: pink

#### Plain Key Value

    env:
      - name: APP_COLOR
        value: pink

#### ConfigMap

    env:
      - name: APP_COLOR
        valueFrom:
          configMapKeyRef:

#### Secrets

    env:
      - name: APP_COLOR
        valueFrom:
          secretKeyRef:

## Section 4:93

### Configuring ConfigMaps in Applications

Manage configuration data centrally using a ConfigMap.

-   create the ConfigMap
-   inject ConfigMap into pod

#### Imperative

    kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MOD=prod

#### Declarative

    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: app-config
    data:
      APP_COLOR: blue
      APP_MODE: prod

Create the configMap
`kubectl create -f config-map.yaml`

View configMaps
`kubectl get configmaps`

#### ConfigMap in Pods

    apiVersion: v1
    kind: Pod
    metadata:
      name: simple-webapp-color
    spec:
      containers:
        - name: simple-webapp-color
          image: simple-webapp-color
          ports:
            - containerPort: 8080
          envFrom:
            - configMapRef:
                  name: app-config

## Section 4:95

### Configure Secrets in Applications
