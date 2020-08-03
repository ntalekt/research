# Lightning Labs Progress: 2 / 2

1.  Upgrade Cluster
    <https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/>

2.  


    kubectl -n admin2406 get deployment -o custom-columns=DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[].image,READY_REPLICAS:.status.readyReplicas,NAMESPACE:.metadata.namespace --sort-by=.metadata.name > /opt/admin2406_data

3.  Change port 2379 to 6443

kubectl cluster-info --kubeconfig /root/admin.kubeconfig

4.

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      labels:
        app: nginx
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1.16
            ports:
            - containerPort: 80

To Upgrade

    kubectl set image deployment/nginx-deployment nginx=nginx:1.17 --record

5.  Create a claim


    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      annotations:
      name: mysql-alpha-pvc
      namespace: alpha
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
      storageClassName: slow

6\.

    ETCDCTL_API=3 etcdctl \
        snapshot save \
        --cacert=/etc/kubernetes/pki/etcd/ca.crt \
        --cert=/etc/kubernetes/pki/etcd/server.crt \
        --key=/etc/kubernetes/pki/etcd/server.key \
        --endpoints=127.0.0.1:2379 \
        /opt/etcd-backup.db

7\.

    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: null
      labels:
        run: secret-1401
      name: secret-1401
      namespace: admin1401
    spec:
      volumes:
      - name: secret-volume
        secret:
          secretName: dotfile-secret
      containers:
      - command:
        - sleep
        args:
        - "4800"
        image: busybox
        name: secret-admin
        volumeMounts:
        - name: secret-volume
          readOnly: true
          mountPath: "/etc/secret-volume"
