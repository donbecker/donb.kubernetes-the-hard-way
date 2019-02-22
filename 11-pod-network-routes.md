* grab internal IP and pod CIDR for the worker and create network route
    * worker-0
        * `$instance="worker-0"; $internalip=$(gcloud compute instances describe ${instance} --format 'value[](networkInterfaces[0].networkIP)'); $podcidr=$(gcloud compute instances describe ${instance} --format 'value[](metadata.items[0].value)'); gcloud compute routes create kubernetes-route-$instance --network kubernetes-the-hard-way --next-hop-address $internalip --destination-range $podcidr`
    * worker-1
        * `$instance="worker-1"; $internalip=$(gcloud compute instances describe ${instance} --format 'value[](networkInterfaces[0].networkIP)'); $podcidr=$(gcloud compute instances describe ${instance} --format 'value[](metadata.items[0].value)'); gcloud compute routes create kubernetes-route-$instance --network kubernetes-the-hard-way --next-hop-address $internalip --destination-range $podcidr`
    * worker-2
        * `$instance="worker-2"; $internalip=$(gcloud compute instances describe ${instance} --format 'value[](networkInterfaces[0].networkIP)'); $podcidr=$(gcloud compute instances describe ${instance} --format 'value[](metadata.items[0].value)'); gcloud compute routes create kubernetes-route-$instance --network kubernetes-the-hard-way --next-hop-address $internalip --destination-range $podcidr`
* verify routes
    * `gcloud compute routes list --filter "network: kubernetes-the-hard-way"`
* next: https://github.com/donbecker/donb.kubernetes-the-hard-way/blob/master/12-dns-addon.md
