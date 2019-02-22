* from the workstation, generate a kubeconfig file for the admin user
    * grab public IP
        * `$KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way --region $(gcloud config get-value compute/region) --format 'value(address)')`
    * set cluster for local kubectl config
        * `kubectl config set-cluster kubernetes-the-hard-way --certificate-authority=ca.pem --embed-certs=true --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443`
    * set key creds for local kubectl config
        * `kubectl config set-credentials admin --client-certificate=admin.pem --client-key=admin-key.pem`
    * set context for local kubectl config
        * `kubectl config set-context kubernetes-the-hard-way --cluster=kubernetes-the-hard-way --user=admin`
    * set active context
        * `kubectl config use-context kubernetes-the-hard-way`
    * verify cluster health
        * `kubectl get componentstatuses`
    * verify node list 
        * `kubectl get nodes`
* next: https://github.com/donbecker/donb.kubernetes-the-hard-way/blob/master/11-pod-network-routes.md
