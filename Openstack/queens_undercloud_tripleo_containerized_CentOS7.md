# TripleO Undercloud Centos7

## Prepare nodes
> 1. Update `/etc/sysconfig/network-scripts/ifcfg-enp0s3` to set static IP

```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s3
UUID=6eee4e43-18ba-4c77-8fee-636856677923
DEVICE=enp0s3
ONBOOT=yes
IPADDR=192.168.1.5
PREFIX=24
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=192.168.1.2
```
> 2. Set hostname

```
sudo hostnamectl set-hostname undercloud.lab
sudo hostnamectl set-hostname --transient undercloud.lab
```
> 3. Update `/etc/hosts`

```
127.0.0.1   undercloud.lab undercloud
```
> 4. Reboot

```
sudo reboot
```
> 5. Add `stack` user

```
sudo useradd -d /home/stack -m stack
sudo passwd stack
```
> 6. Create sudoers file for stack and edit

```
sudo echo "stack ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/stack
sudo chmod 0440 /etc/sudoers.d/stack
sudo sed -i s'/Defaults requiretty/#Defaults requiretty'/g /etc/sudoers
```
> 7. Change to `stack` user

```
su - stack
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

> 9. Update

```
sudo yum -y update && yum -y upgrade
```

> 10. Install and Configure NTP

```
sudo yum install ntp ntpdate ntp-doc -y
sudo ntpdate 0.us.pool.ntp.org
sudo hwclock --systohc
sudo systemctl enable ntpd.service
sudo systemctl start ntpd.service
```
> /etc/ntp.conf

```
driftfile /var/lib/ntp/drift
restrict 0.0.0.0 mask 0.0.0.0

server 0.us.pool.ntp.org iburst
server 1.us.pool.ntp.org iburst
server 2.us.pool.ntp.org iburst
server 3.us.pool.ntp.org iburst
```
> 11. Enable needed repositories

```
sudo yum install -y yum-utils
sudo yum-config-manager --enable rhelosp-rhel-7-server-opt
```

> 12. Download and install the python2-tripleo-repos rpm

```
sudo yum install -y https://trunk.rdoproject.org/centos7/current/python2-tripleo-repos-0.0.1-0.20180806134256.86aee5f.el7.noarch.rpm
```

> 13. Enable the current Queens and Ceph repositories

```
sudo -E tripleo-repos -b queens current ceph
```

> 14. Install the TripleO client

```
sudo yum install -y python-tripleoclient
```

> 15. If you intend to deploy Ceph in the overcloud, or configure the overcloud to use an external Ceph cluster, and are running Pike or newer, then install ceph-ansible on the undercloud

```
sudo yum install -y ceph-ansible
```

> 16. Prepare the configuration file

```
sudo cp /usr/share/python-tripleoclient/undercloud.conf.sample ~/undercloud.conf
```

```
[DEFAULT]
local_ip = 192.168.1.4/24
undercloud_public_vip = 192.168.1.10
undercloud_admin_vip = 192.168.1.11
local_interface = enp0s8
masquerade_network = 192.168.1.0/24
dhcp_start = 192.168.1.20
dhcp_end = 192.168.1.30
network_cidr = 192.168.1.0/24
network_gateway = 192.168.1.4
inspection_iprange = 192.168.1.50,192.168.1.80
undercloud_nameservers = 8.8.8.8
```

## Deploy undercloud

```
openstack undercloud install
```

## Confirm Undercloud

```
. /home/stack/stackrc
openstack endpoint list
openstack host list
python -m json.tool /etc/os-net-config/config.json
sudo ovs-vsctl show
```

## Configure the Undercloud

```
sudo yum install rhosp-director-images rhosp-director-images-ipa
```

## Reference
* https://access.redhat.com/documentation/en-us/reference_architectures/2017/html/deploying_red_hat_openstack_platform_10_on_hpe_proliant_dl_servers/deploying_the_undercloud_and_overcloud
* https://docs.openstack.org/tripleo-docs/latest/install/installation/installing.html
* https://docs.openstack.org/tripleo-quickstart/latest/basic-usage.html#tripleo-quickstart
* https://www.youtube.com/watch?v=lv233gPynwk
* https://www.youtube.com/watch?v=ulpxlNFfbF8
* https://redhatstackblog.redhat.com/2018/07/10/virtualize-your-openstack-control-plane-with-red-hat-virtualization-and-red-hat-openstack-platform-13/
