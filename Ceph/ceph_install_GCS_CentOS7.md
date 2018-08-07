# ceph install on GCS Centos7
ceph 10.2.11
## Prepare nodes

> 1. Open GCS Cloud Shell
> 2. Connect to a node

```
gcloud compute --project "ceph-211918" ssh --zone "us-west2-a" "admin-node"
```
> 3. Add new user (on each node)

```
sudo useradd -d /home/cephuser -m cephuser
sudo passwd cephuser
```
> 4. Create sudoers file for cephuser and edit

```
sudo echo "cephuser ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephuser
sudo chmod 0440 /etc/sudoers.d/cephuser
sudo sed -i s'/Defaults requiretty/#Defaults requiretty'/g /etc/sudoers
```
> 5. Check the second disk (on data nodes)

```
sudo fdisk -l /dev/sdb
```
> 6. Partition (on data nodes)

```
sudo parted -s /dev/sdb mklabel gpt mkpart primary xfs 0% 100%
sudo mkfs.xfs /dev/sdb -f
sudo blkid -o value -s TYPE /dev/sdb
```
## Install ceph-deploy on the admin-node
> 1. Change to `cephuser`, add the ceph repo and install the Ceph deployment tool 'ceph-deploy' with the yum command.

```
su - cephuser
sudo rpm -Uhv http://download.ceph.com/rpm-jewel/el7/noarch/ceph-release-1-1.el7.noarch.rpm
sudo yum update -y && sudo yum install ceph-deploy -y
```
## Create New Cluster Config
> 1. Create a new cluster directory
```
mkdir cluster
cd cluster/
```
> 2. Next, create a new cluster configuration with the 'ceph-deploy' command, define the monitor node to be 'node1'.
```
ceph-deploy new node1
```
## Install Ceph on All Nodes
> 1. Install ceph on all the nodes
```
ceph-deploy install admin-node node1 node2 node3
```
> 2. Deploy the ceph-mon on node1 node.
```
ceph-deploy mon create-initial
```
> 3. Create the monitor key
```
ceph-deploy gatherkeys node1
```
## Adding OSDs to the Cluster
> 1. Install ceph on all the nodes
```
ceph-deploy disk list node1 node2 node3
```
> 2. Next, delete the /dev/sdb partition tables on all nodes with the zap option.
```
ceph-deploy disk zap node1:/dev/sdb node2:/dev/sdb node3:/dev/sdb
```
> 3. Now prepare all OSD nodes. Make sure there are no errors in the results.
```
ceph-deploy osd prepare node1:/dev/sdb node2:/dev/sdb node3:/dev/sdb
```
> 4. Activate the OSDs
```
ceph-deploy osd activate node1:/dev/sdb1 node2:/dev/sdb1 node3:/dev/sdb1
```
> 5. Check the status
```
ceph-deploy disk list node1 node2 node3
```
> 6. Next, deploy the management-key to all associated nodes.
```
ceph-deploy admin admin-node node1 node2 node3
```
> 7. Change the permission of the key file by running the command below on all nodes.
```
sudo chmod 644 /etc/ceph/ceph.client.admin.keyring
```
# Expanding Cluster
## Add Metadata Server
> 1. To use CephFS, you need at least one metadata server. Execute the following to create a metadata server
```
ceph-deploy mds create node1
```
## Adding Monitors
A Ceph Storage Cluster requires at least one Ceph Monitor and Ceph Manager to run. For high availability, Ceph Storage Clusters typically run multiple Ceph Monitors so that the failure of a single Ceph Monitor will not bring down the Ceph Storage Cluster. Ceph uses the Paxos algorithm, which requires a majority of monitors (i.e., greather than N/2 where N is the number of monitors) to form a quorum. Odd numbers of monitors tend to be better, although this is not required.
> 1. Add two Ceph Monitors to your cluster:
Add two Ceph Monitors to your cluster:
```
ceph-deploy mon add node2 node3
ceph quorum_status --format json-pretty
```
## Testing
> 1. From the admin-node, log in to the ceph monitor server 'node1'.
```
ssh node1
```
> 2. Run the command below to check the cluster health.
```
sudo ceph health
```
> 3. Now check the cluster status.
```
sudo ceph -s
sudo ceph osd tree
```
## Reference
http://docs.ceph.com/docs/master/start/
https://access.redhat.com/documentation/en/red-hat-ceph-storage/
https://www.howtoforge.com/tutorial/how-to-build-a-ceph-cluster-on-centos-7/
