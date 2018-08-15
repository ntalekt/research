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
> 7. Disable SELINUX

```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```
> 8. Disable firewalld and NetworkManager

```
sudo systemctl disable firewalld
sudo systemctl stop firewalld
sudo systemctl disable NetworkManager
sudo systemctl stop NetworkManager
sudo systemctl enable network
sudo systemctl start network
```

> 9. Ensure that package manager has priority/preferences installed and enabled.

```
sudo yum install yum-plugin-priorities -y
```

>10. Install and Configure NTP

```
yum install ntp ntpdate ntp-doc -y
ntpdate 0.us.pool.ntp.org
hwclock --systohc
systemctl enable ntpd.service
systemctl start ntpd.service
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
> 1. Update ceph.conf

```
[global]
fsid = 9c36b53a-434b-4187-99d5-4bd550042285
mon_initial_members = node1,node2,node3
mon_host = 192.168.1.7,192.168.1.8,192.168.1.9
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
public network = 192.168.1.0/24
osd pool default size = 2
```

> 2. Add two Ceph Monitors to your cluster:

```
ceph-deploy --overwrite-conf config push admin-node node1 node2 node3
ceph-deploy mon create node2 node3
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

## Upgrade Jewel to Luminous
> 1. Login to admin node and ensure the sortbitwise flag is enabled.
```
ceph osd set sortbitwise
```
> 2. Set the noout flag for the duration of the upgrade. This will instruct Ceph to do not rebalance the cluster.
```
ceph osd set noout
```
> 3. Update yum repo to target the Luminous release
```
sed -i 's/jewel/luminous/' /etc/yum.repos.d/ceph.repo
```
> 4. Update ceph-deploy
```
sudo yum install ceph-deploy python-pushy
```
> 5. Update the admin-node
```
ceph-deploy install --release luminous ceph
```
> 6. Upgrade the monitors
```
ceph-deploy install --release luminous node1 node2 node3
```
> 7. Restart the monitor service on each monitor node
```
systemctl restart ceph-mon.target
```
> 8. Deploy mgr nodes
```
ceph-deploy mgr create node1 node2 node3
```
> 9. Upgrade the OSDs
```
ceph-deploy install --release luminous node1 node2 node2
```
> 10. Restart the OSD service on each OSD
```
systemctl restart ceph-osd.target
```
> 11. Check versions
```
ceph versions
```
> 12. The upgrade is complete. Now disallow any pre-Luminous OSD and enable Luminous-only functionality.
```
ceph osd require-osd-release luminous
```
> 13. Disable the noout options so that the cluster can re-balance when needed.
```
ceph osd unset noout
```

## Enable dashboard
> 1. Enable module on mgr nodes (/etc/ceph/ceph.conf)
```
[mgr]
mgr_modules = dashboard
```
> 2. Setting server and port
```
ceph config-key put mgr/dashboard/server_addr ::
```
> 3. Login to admin node and restart mgr services
```
sudo systemctl restart ceph-mgr@node1
sudo systemctl restart ceph-mgr@node2
sudo systemctl restart ceph-mgr@node3
```
> 4. Enable the dashboard
```
ceph mgr module enable dashboard
```
> 5. Browse to “http://active_mgr_host:7000/“

## Reference
* http://docs.ceph.com/docs/master/start/
* https://access.redhat.com/documentation/en/red-hat-ceph-storage/
* https://www.howtoforge.com/tutorial/how-to-build-a-ceph-cluster-on-centos-7/
* https://www.virtualtothecore.com/en/upgrade-ceph-cluster-luminous/
