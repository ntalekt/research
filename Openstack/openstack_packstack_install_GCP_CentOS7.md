## OpenStack (Packstack) Install on GCS (RHEL/CentOS)
https://github.com/openstack/packstack
### Requirements
> * Single server (16GB) memory
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

#### Static IP creation
* Create a new static IP in same region as instance [Reference](https://cloud.google.com/compute/docs/ip-addresses/reserve-static-external-ip-address)
```
gcloud compute addresses create openstack --region us-west2 --ip-version IPV4
```
* List static IPs in project
```
gcloud compute addresses list
```

#### Instance deployment
##### gcloud sdk
* Deploy instance from image (Update [IP_ADDRESS] from previous step) - [Reference](https://cloud.google.com/compute/docs/instances/create-start-instance)
```
gcloud compute instances create openstack --zone us-west2-a --machine-type=n1-highmem-2 --image nested-vm-image --boot-disk-size=10GB --tags http-server --address [IP_ADDRESS]
```
* Connect to the instance - [Reference](https://cloud.google.com/compute/docs/instances/connecting-to-instance)
```
gcloud compute ssh --zone "us-west2-a" "openstack"
```

##### instance shell
* Set root password
```
sudo passwd root
```
* Confirm virtualization extensions are enabled. We should get the word either vmx or svm in the output, otherwise CPU doesn’t support virtualization.
```
grep -E '(vmx|svm)' /proc/cpuinfo
```
* Edit sshd and enable root access. Packstack needs this to deploy
```
sudo vim /etc/ssh/sshd_config
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

## Install RDO repo
* Update yum
```
sudo yum update -y
```
* Install RDO repo rpm
```
sudo yum install -y https://rdoproject.org/repos/rdo-release.rpm
```
* Update yum
```
sudo yum update -y
```

## Install OpenStack Installer (Packstack)
* To install the packstack installer run
```
sudo yum install -y openstack-packstack
```

## Install OpenStack
* To put everything on a single instance run
```
sudo packstack --allinone --provision-demo=n
```

## Configure Horizon for external access
* We must add an alias to the instances external IP address and restart httpd in order to access Horizon from public network.
```
sudo vim /etc/httpd/conf.d/15-horizon_vhost.conf
```
Add a new server alias of your instance external IP
`ServerAlias [IP_ADDRESS]`

* Restart httpd
```
sudo systemctl restart httpd
```
* Open a browser and point it towards Horizon http://[IP_ADDRESS]/dashboard
* Username and password are in /root/keystonerc_admin
```
sudo cat /root/keystonerc_admin
```
