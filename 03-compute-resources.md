* create VPC network
    * `gcloud compute networks create kubernetes-the-hard-way --subnet-mode custom`
* create subnet
    * `gcloud compute networks subnets create kubernetes --network kubernetes-the-hard-way --range 10.240.0.0/24`
* create firewall rule for internal traffic
    * `gcloud compute firewall-rules create kubernetes-the-hard-way-allow-internal --allow tcp,udp,icmp --network kubernetes-the-hard-way --source-ranges 10.240.0.0/24,10.200.0.0/16`
* create firewall rule for external SSH, ICMP and HTTPS
    * `gcloud compute firewall-rules create kubernetes-the-hard-way-allow-external --allow tcp:22,tcp:6443,icmp --network kubernetes-the-hard-way --source-ranges 0.0.0.0/0`
* list firewalls in VPC network
    * `gcloud compute firewall-rules list --filter="network:kubernetes-the-hard-way"`
* allocate static public IP address for the external load balancer in front of the K8s API Servers
    * `gcloud compute addresses create kubernetes-the-hard-way --region $(gcloud config get-value compute/region)`
* verify IP address was created
    * `gcloud compute addresses list --filter="name=('kubernetes-the-hard-way')"`
* create controllers: three compute instances for K8s control plane
    * `gcloud compute instances create controller-0 --private-network-ip 10.240.0.10 --async --boot-disk-size 200GB --can-ip-forward --image-family ubuntu-1804-lts --image-project ubuntu-os-cloud --machine-type n1-standard-1 --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring --subnet kubernetes --tags kubernetes-the-hard-way,controller`
    * `gcloud compute instances create controller-1 --private-network-ip 10.240.0.11 --async --boot-disk-size 200GB --can-ip-forward --image-family ubuntu-1804-lts --image-project ubuntu-os-cloud --machine-type n1-standard-1 --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring --subnet kubernetes --tags kubernetes-the-hard-way,controller`
    * `gcloud compute instances create controller-2 --private-network-ip 10.240.0.12 --async --boot-disk-size 200GB --can-ip-forward --image-family ubuntu-1804-lts --image-project ubuntu-os-cloud --machine-type n1-standard-1 --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring --subnet kubernetes --tags kubernetes-the-hard-way,controller`
* create workers: three compute instances to server as K8s worker nodes
    * `gcloud compute instances create worker-0 --private-network-ip 10.240.0.20 --metadata pod-cidr=10.200.0.0/24 --async --boot-disk-size 200GB --can-ip-forward --image-family ubuntu-1804-lts --image-project ubuntu-os-cloud --machine-type n1-standard-1 --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring --subnet kubernetes --tags kubernetes-the-hard-way,worker`
    * `gcloud compute instances create worker-1 --private-network-ip 10.240.0.21 --metadata pod-cidr=10.200.1.0/24 --async --boot-disk-size 200GB --can-ip-forward --image-family ubuntu-1804-lts --image-project ubuntu-os-cloud --machine-type n1-standard-1 --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring --subnet kubernetes --tags kubernetes-the-hard-way,worker`
    * `gcloud compute instances create worker-2 --private-network-ip 10.240.0.22 --metadata pod-cidr=10.200.2.0/24 --async --boot-disk-size 200GB --can-ip-forward --image-family ubuntu-1804-lts --image-project ubuntu-os-cloud --machine-type n1-standard-1 --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring --subnet kubernetes --tags kubernetes-the-hard-way,worker`
* list instances
    * `gcloud compute instances list`
* verify SSH to all instances (spawns Putty on Windows)
    * `gcloud compute ssh controller-0`
    * `gcloud compute ssh controller-1`
    * `gcloud compute ssh controller-2`
    * `gcloud compute ssh worker-0`
    * `gcloud compute ssh worker-1`
    * `gcloud compute ssh worker-2`
