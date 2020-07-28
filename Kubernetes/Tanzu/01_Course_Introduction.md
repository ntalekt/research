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
    bosh upload-stemcell --sha1 8a91be2437e1c37991e601c0869e4d95a023f704

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
