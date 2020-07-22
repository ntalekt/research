# Storage Progress: 13 / 13

## Section 6:152

### Storage in Docker

Default stores at `/var/lib/docker`

Layered architecture in dockerfile

## Section 6:153

### Volume Driver Plugins in Docker

-   Storage drivers - AUFS, ZFS, BTRFS, etc.
-   Volume drivers - local, Azure file storage, Convoy, GlusterFS, NetApp, VMware vSphere Storage, etc.

Volumes are not controlled by storage drivers, they are controlled by volume drives.

## Section 6:154

### Container Storage Interface (CSI)

Standard that defines how an orchestration engine, like Kubernetes, would communicate with multiple storage solutions, like portworx, Amazon EBS, Dell EMC, etc.

## Section 6:156

### Volumes

Pods created in Kubernetes pods are transient but you can persist data by attaching volume.

    apiVersion: v1
    kind: Pod
    metadata:
      name: random-number-generator
    spec:
      containers:
      - image: alpine
        name: alpine
        command: ["/bin/sh","-c"]
        args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
        volumeMounts:
        - mountPath: /opt
          name: data-volume
      volumes:
      - name: data-volume
        hostPath:
          path: /data
          type: Directory
      - name: data-volume2
        awsElasticBlockStore:
          volumeID: <volume-id>
          fsType: ext4

## Section 6:157

### Persistent Volumes

Would need to make storage on each pod. Persistent volumes allows administrator to create a cluster wide pool of storage.

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: pv-log
    spec:
      accessModes:
        - ReadWriteMany
      capacity:
        storage: 100Mi
      hostPath:
        path: /pv/log

## Section 6:158

### Persistent Volumes Claim

Every persistent volume claim is bound to a single persistent volume. 1:1 relationship to persistent volumes and persistent volume claims.

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: claim-log-1
    spec:
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 50Mi
