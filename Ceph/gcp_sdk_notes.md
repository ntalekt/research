> * List projects
```
gcloud projects list
```
> * list instances
```
gcloud compute instances list
```
> * ssh to instance
```
gcloud compute --project "ceph-211918" ssh --zone "us-west2-a" "node2"
```
> * Create new image with vt enabled
```
gcloud compute images create nested-vm-image --source-image centos-7 --licenses "https://www.googleapis.com/compute/v1/projects/vm-options/global/licenses/enable-vmx"
```
> * Deploy instance from image
```
gcloud compute instances create example-nested-vm --zone us-west2-a --machine-type=f1-micro --image nested-vm-image --boot-disk-size=10GB
```

References
https://www.linuxtechi.com/install-kvm-hypervisor-on-centos-7-and-rhel-7/
https://www.howtoforge.com/vnc-server-installation-on-centos-7
