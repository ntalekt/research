# Advanced BOSH

## Deploy a distributed system

### Deploy Zookeeper

Upload Zookeeper release to director

    bosh upload-release --sha1 a6d227abceebf1e3e68ce4a3cabf68b0b93165d2 \
      https://bosh.io/d/github.com/cppforlife/zookeeper-release?v=0.0.10

Grab the manifest

    git clone https://github.com/cppforlife/zookeeper-release.git

Update instance, vm_type, networks, and AZS

    instances: 3
    vm_type: small
    networks:
    - name: services
    azs: [us-central1-a, us-central1-b, us-central1-c]

Deploy zookeeper from modified manifest

    bosh -d zookeeper deploy manifests/zookeeper.yml

Run status errand

    bosh -d zookeeper run-errand status

ssh to instance

    bosh ssh -d zookeeper zookeeper/88ead7e0-b476-4274-b0ab-df00b9d9b666

Connect to zookeeper cli and create some data

    exec /var/vcap/jobs/zookeeper/bin/ctl cli
    create /rick_test my_data

ssh to a different instance

    bosh ssh -d zookeeper zookeeper/a265b3ed-dff3-4354-9815-c33961613a5a

Connect to zookeeper cli and ls

    exec /var/vcap/jobs/zookeeper/bin/ctl cli
    ls /

## Deploy Concourse

Export cloud config

    bosh cloud-config > cloud-config.yml

Clone the concourse repo

    git clone https://github.com/concourse/concourse-bosh-deployment.git

List repos and checkout version

    git tag --list
    git checkout v5.8.1

Create variables file

    vi concourse-vars-file.yml

Contents

    ---
    deployment_name: concourse
    network_name: services
    web_vm_type: small.disk
    db_vm_type: small.disk
    worker_vm_type: small.disk
    db_persistent_disk_type: 10GB
    azs: [us-central1-a, us-central1-b, us-central1-c]
    external_url: https://35.225.118.33
    external_host: 35.225.118.33
    web_instances: 1
    worker_instances: 2
    local_user:
      username: admin
      password: admin
    web_network_vm_extension: lb
    web_network_name: services

Update cloud config vm_type

        - cloud_properties:
            cpu: 1
            ram: 2048
            root_disk_size_gb: 50
            root_disk_type: pd-standard
            tags:
            - rick
          name: small.disk

Add lb under vm_extensions:

    - cloud_properties:
        target_pool: pks-0728-<LB>-concourse
      name: lb

add 10GB under disk_types:

    - cloud_properties:
        type: pd-standard
      disk_size: 10240
      name: "10GB"

Upload cloud config

    bosh update-cloud-config cloud-config.yml

deploy

    bosh deploy --deployment concourse concourse.yml \
      --vars-file ../versions.yml \
      --vars-file concourse-vars-file.yml \
      --ops-file operations/basic-auth.yml \
      --ops-file operations/privileged-http.yml \
      --ops-file operations/privileged-https.yml \
      --ops-file operations/tls.yml \
      --ops-file operations/tls-vars.yml \
      --ops-file operations/web-network-extension.yml \
      --ops-file operations/scale.yml
