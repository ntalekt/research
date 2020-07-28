# TKGI Fundamentals Overview

VMware Tanzu Kubernetes Grid

## Kubernetes 101

-   Spin up a GKE Cluster
-   Deploy an App to Kubernetes

## BOSH

-   Concepts

## TKGI

-   Provision Ops Manager & BOSH Director
-   Provision TKGI
-   Deploy a K8S Cluster & Configure Cluster Load Balancer
-   Install Harbor Registry & Exercise Harbor
-   Deploy an application to K8S
-   Helm

## Lab - Kubernetes Overview

-   Kubernetes cluster hosted on GCP
-   <https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app>

## Presentation - Introduction to BOSH

When it comes to Kubernetes cluster BOSH will be phased out and ClusterAPI will be phased in.

BOSH does many things:

-   Provisions VMs in the cloud
-   Installs and configures packages, starts services
-   Monitors processes and VMs (monit Agent)

Inputs:

-   Stemcell - Image (prebuild with bosh, monit, and runit agents) <https://bosh.io/stemcells/>
-   Releases - Binaries on the blobstore
-   Deployment Manifests (name, stemcells, releases, instance_groups) - Deployment manifests are cloud agnostic.

## Lab - BOSH - Deploy a simple release

Make sure to install the netcat-openbsd version of netcat with `apt install netcat-openbsd`.

Install BOSH CLI

    wget https://github.com/cloudfoundry/bosh-cli/releases/download/v6.3.1/bosh-cli-6.3.1-linux-amd64
    mv bosh-cli-6.3.1-linux-amd64 bosh
    chmod +x bosh
    mkdir bin
    mv bosh bin/
    export PATH="/home/rickrocklin/bin:$PATH"

Source the environment file instructor provided

    source test-env
    bosh env

Upload a release

    bosh upload-release --sha1 bf156c545c45ed4e57563274f91dbd433000d776 \
          https://bosh.io/d/github.com/cloudfoundry-community/nginx-release?v=1.13.12

Upload a stemcell

    wget https://bosh-gce-light-stemcells.s3-accelerate.amazonaws.com/621.61/light-bosh-stemcell-621.61-google-kvm-ubuntu-xenial-go_agent.tgz
    bosh upload-stemcell --sha1 8a91be2437e1c37991e601c0869e4d95a023f704 light-bosh-stemcell-621.61-google-kvm-ubuntu-xenial-go_agent.tgz

Deploy

    bosh deploy --deployment nginx --non-interactive ~/manifests/nginx.yaml

Check deployment and instance name

    bosh deployments
    bosh instances

SSH to instance and issue curl command

    bosh -d nginx ssh nginx/9bb57fd2-7e07-49a6-8050-7d331a277987 -c 'curl http://localhost'

## Lab - BOSH - Inside the VM

SSH to the instance and change to root

    bosh -d nginx ssh nginx/9bb57fd2-7e07-49a6-8050-7d331a277987
    sudo -i

Get logs from instance

    bosh -d nginx logs nginx/9bb57fd2-7e07-49a6-8050-7d331a277987

File locations

-   nginx binary: `/var/vcap/packages/nginx/sbin/nginx`
-   nginx config file: `/var/vcap/jobs/nginx/etc/nginx.conf`
-   nginx access logs: `/var/vcap/sys/log/nginx/nginx-access.log`
-   Index html file: `/var/vcap/store/nginx/index.html`
-   BOSH agent log file: `/var/vcap/bosh/log/current`

Observe bosh-agent querying monit for its status using

    grep --after-context 4 'monit status' /var/vcap/bosh/log/current

Force a bosh-agent to restart by killing it. Use grep to filter its logs.

    killall bosh-agent && \
     grep --after-context 500 'Starting agent' /var/vcap/bosh/log/current | \
     grep DEBUG | grep --invert-match 'Cmd Runner\|File System'

Observe how the monit status output is reflected in the heartbeat

    grep --after-context 3 "Sending hm message 'heartbeat'" \
        /var/vcap/bosh/log/current

Force nginx to restart by killing it and grep filters

    killall nginx && \
     grep --after-context 3 "Sending hm message 'alert'" \
     /var/vcap/bosh/log/current

## Lab - BOSH - Persistent Disks

SSH to the instance and change to root

    bosh -d nginx ssh nginx/9bb57fd2-7e07-49a6-8050-7d331a277987
    sudo -i

Check disks

    df -h

Modify index.html

    vi /var/vcap/store/nginx/index.html
    curl localhost

Stop bosh agent to force BOSH resurrector to replace with a new one

    sv stop agent
    bosh tasks

SSH back in and check if index.html persisted

    bosh -d nginx ssh nginx/9bb57fd2-7e07-49a6-8050-7d331a277987
    curl localhost

Add persistent disk to manifest

    persistent_disk: 1024

Redeploy

    bosh deploy --deployment nginx --non-interactive ~/manifests/nginx.yaml

SSH and look to confirm new disk (/dev/sdb1)

    bosh -d nginx ssh nginx/9bb57fd2-7e07-49a6-8050-7d331a277987
    df -h

Modify index.html

    vi /var/vcap/store/nginx/index.html
    curl localhost

Stop bosh agent to force BOSH resurrector to replace with a new one

    sv stop agent
    bosh tasks

SSH confirm index persisted

    bosh -d nginx ssh nginx/9bb57fd2-7e07-49a6-8050-7d331a277987
    curl localhost

### Resize disk

Resize persistent disk in manifest and redeploy

    persistent_disk: 2048
    bosh deploy --deployment nginx --non-interactive ~/manifests/nginx.yaml

Confirm data persisted the disk resize

    bosh -d nginx ssh nginx/9bb57fd2-7e07-49a6-8050-7d331a277987
    curl localhost
    df -h

## Lab - BOSH - Patch the OS

    wget https://bosh-gce-light-stemcells.s3-accelerate.amazonaws.com/621.64/light-bosh-stemcell-621.64-google-kvm-ubuntu-xenial-go_agent.tgz
    bosh upload-stemcell --sha1 d0ae2cfe2c8ae4c4fd058bbf50a7c4d97af9dc1d light-bosh-stemcell-621.64-google-kvm-ubuntu-xenial-go_agent.tgz

Edit the manifest to use the newer version and redeploy

    vi nginx.yaml
    bosh deploy --deployment nginx --non-interactive ~/manifests/nginx.yaml

## Lab - BOSH - Patch the OS

Upload a new release

    bosh upload-release --sha1 13cf87b2394c7d3924f9d66836c56302fb46a90d \
      https://bosh.io/d/github.com/cloudfoundry-community/nginx-release?v=1.17.0

Edit the manifest to use the newer release version and redeploy

    vi nginx.yaml
    bosh deploy --deployment nginx --non-interactive ~/manifests/nginx.yaml
