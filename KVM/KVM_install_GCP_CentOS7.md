## KVM Install on GCP (RHEL/CentOS)
### Requirements
> * Single server (4GB) memory
> * CentOS 7
>  * As tested: Linux release 7.5.1804 (Core) - `cat /etc/redhat-release`
> * At least one NIC
> * Virtualization extensions enabled

### Deployment
#### Image creation
* Create new CentOS image with virtualization extensions enabled - [Reference](https://cloud.google.com/compute/docs/instances/enable-nested-virtualization-vm-instances)
```
gcloud compute images create nested-vm-image --source-image-family=centos-7 --source-image-project=centos-cloud --licenses "https://www.googleapis.com/compute/v1/projects/vm-options/global/licenses/enable-vmx"
```

#### Instance deployment
##### gcloud sdk
* Deploy instance from image - [Reference](https://cloud.google.com/compute/docs/instances/create-start-instance)
```
gcloud compute instances create kvm1 --zone us-west2-a --machine-type=g1-small --image nested-vm-image --boot-disk-size=10GB
```
* Connect to the instance - [Reference](https://cloud.google.com/compute/docs/instances/connecting-to-instance)
```
gcloud compute ssh --zone "us-west2-a" "kvm1"
```

##### instance shell
* Set root password
```
sudo passwd root
```
* Confirm virtualization extensions are enabled. We should get the word either vmx or svm in the output, otherwise CPU doesn’t support virtualization.
```
lscpu | grep Virtualization
```
* Edit sshd and enable root access and password authentication. Packstack needs this to deploy
```
sudo vim /etc/ssh/sshd_config
PermitRootLogin yes
PasswordAuthentication yes
sudo systemctl restart sshd
```
* Disable firewalld and NetworkManager
```
sudo systemctl disable firewalld
sudo systemctl stop firewalld
sudo systemctl disable NetworkManager
sudo systemctl stop NetworkManager
sudo systemctl enable network
sudo systemctl start network
```

## Install KVM
* Update yum and install KVM
```
sudo yum update -y
sudo yum install qemu-kvm libvirt libvirt-python libguestfs-tools virt-install -y
```
* Start the libvirtd service
```
sudo systemctl enable libvirtd
sudo systemctl start libvirtd
```

## Verify kvm installation
Make sure KVM module loaded using lsmod command and grep command
```
lsmod | grep -i kvm
```

## Configure bridged networking
* By default dhcpd based network bridge configured by libvirtd. You can verify that with the following commands
```
sudo brctl show
sudo virsh net-list
```

* All VMs only have network access to other VMs on the same server. A private network 192.168.122.0/24 is created for you. Verify it
```
sudo virsh net-dumpxml default
```

* If you want your VMs avilable to other servers on your LAN, setup a a network bridge on the server that connected to the your LAN. Update your nic config file such as ifcfg-eth0
```
sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
 * Add line
 ```
 BRIDGE=br0
 ```
 * Create new ifcfg-br0
 ```
 sudo vi /etc/sysconfig/network-scripts/ifcfg-br0
 ```
   * Add line
 ```
DEVICE="br0"
# I am getting ip from DHCP server #
BOOTPROTO="dhcp"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
ONBOOT="yes"
TYPE="Bridge"
DELAY="0"
 ```
* Restart the networking service
```
sudo systemctl restart NetworkManager
```
* Verify it with brctl command
```
sudo brctl show
```
