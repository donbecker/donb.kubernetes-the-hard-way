* delete external load balancer
    * `gcloud -q compute forwarding-rules delete kubernetes-forwarding-rule --region $(gcloud config get-value compute/region)`
    * `gcloud -q compute target-pools delete kubernetes-target-pool`
    * `gcloud -q compute http-health-checks delete kubernetes`
    * `gcloud -q compute addresses delete kubernetes-the-hard-way`
* delete firewall rules
    * `gcloud -q compute firewall-rules delete kubernetes-the-hard-way-allow-nginx-service kubernetes-the-hard-way-allow-internal kubernetes-the-hard-way-allow-external kubernetes-the-hard-way-allow-health-check`
* delete vpc
    * `gcloud -q compute routes delete kubernetes-route-worker-0 kubernetes-route-worker-1 kubernetes-route-worker-2`
    * `gcloud -q compute instances delete controller-0 controller-1 controller-2 worker-0 worker-1 worker-2`
    * `gcloud -q compute networks subnets delete kubernetes`
    * `gcloud -q compute networks delete kubernetes-the-hard-way`

