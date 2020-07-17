# Application Lifecycle Management Progress: 23 / 23

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

Don't store passwords in configMaps. Stored in secrets as they are hashed.

#### Create Secrets

##### Imperative

    kubectl create secret generic app-secret --from-literal=DB_Host=mysql

    kubectl create secret generic app-secret --from-file=app_secret.properties

##### Declarative

Values must be encoded using `echo -n 'mysql' | base64`

    apiVersion: v1
    kind: Secret
    metadata:
      name: app-config
    data:
      DB_Host: bXlzcWw=
      DB_User: cm9vdA==
      DB_Password: cGFzc3dyZA==

View Secrets `kubectl get secrets`
View contents of secrets `kubectl get secret app-secret -o yaml`

Decode the hash values `echo -n 'bXlzcWw=' | base64 --decode`

Configure secrets with Pods

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
          - secretRef:
              name: test-secret

## Section 4:99

### Multi Container Pods

Add another container to the array

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
        - name: log-agent
          image: log-agent

Example lab

    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        name: app
      name: app
      namespace: elastic-stack
    spec:
      containers:
        - image: kodekloud/event-simulator
          name: app
          volumeMounts:
          - mountPath: /log
            name: log-volume
        - image: kodekloud/filebeat-configured
          name: sidecar
          volumeMounts:
          - mountPath: /var/log/event-simulator/
            name: log-volume    
      volumes:
      - hostPath:
          path: /var/log/webapp
          type: DirectoryOrCreate
        name: log-volume

## Section 4:102

### InitContainers

Example

    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: busybox:1.28
        command: ['sh', '-c', 'echo The app is running! && sleep 3600']
      initContainers:
      - name: init-myservice
        image: busybox
        command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ; done;']
