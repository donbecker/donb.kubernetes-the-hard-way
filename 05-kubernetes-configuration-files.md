* grab public IP
    * `$KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way --region $(gcloud config get-value compute/region) --format 'value(address)')`
* generate kubeconfig files for worker nodes
    * worker-0
        * `$INSTANCE="worker-0"; kubectl config set-cluster kubernetes-the-hard-way --certificate-authority=".\ca.pem" --embed-certs=true --server="https://${KUBERNETES_PUBLIC_ADDRESS}:6443" --kubeconfig="${instance}.kubeconfig"; kubectl config set-credentials system:node:${instance} --client-certificate="${instance}.pem" --client-key=".\${instance}-key.pem" --embed-certs=true --kubeconfig="${instance}.kubeconfig"; kubectl config set-context default --cluster=kubernetes-the-hard-way --user=system:node:${instance} --kubeconfig=".\${instance}.kubeconfig"; kubectl config use-context default --kubeconfig="${instance}.kubeconfig"`
    * worker-1
        * `$INSTANCE="worker-1"; kubectl config set-cluster kubernetes-the-hard-way --certificate-authority=".\ca.pem" --embed-certs=true --server="https://${KUBERNETES_PUBLIC_ADDRESS}:6443" --kubeconfig="${instance}.kubeconfig"; kubectl config set-credentials system:node:${instance} --client-certificate="${instance}.pem" --client-key=".\${instance}-key.pem" --embed-certs=true --kubeconfig="${instance}.kubeconfig"; kubectl config set-context default --cluster=kubernetes-the-hard-way --user=system:node:${instance} --kubeconfig=".\${instance}.kubeconfig"; kubectl config use-context default --kubeconfig="${instance}.kubeconfig"`
    * worker-2
        * `$INSTANCE="worker-2"; kubectl config set-cluster kubernetes-the-hard-way --certificate-authority=".\ca.pem" --embed-certs=true --server="https://${KUBERNETES_PUBLIC_ADDRESS}:6443" --kubeconfig="${instance}.kubeconfig"; kubectl config set-credentials system:node:${instance} --client-certificate="${instance}.pem" --client-key=".\${instance}-key.pem" --embed-certs=true --kubeconfig="${instance}.kubeconfig"; kubectl config set-context default --cluster=kubernetes-the-hard-way --user=system:node:${instance} --kubeconfig=".\${instance}.kubeconfig"; kubectl config use-context default --kubeconfig="${instance}.kubeconfig"`
    * verify files generated
        * worker-0.kubeconfig
        * worker-1.kubeconfig
        * worker-2.kubeconfig
* generate kubeconfig file for kube-proxy service
    * `$INSTANCE="kube-proxy"; kubectl config set-cluster kubernetes-the-hard-way --certificate-authority=".\ca.pem" --embed-certs=true --server="https://${KUBERNETES_PUBLIC_ADDRESS}:6443" --kubeconfig="${instance}.kubeconfig"; kubectl config set-credentials system:${instance} --client-certificate="${instance}.pem" --client-key=".\${instance}-key.pem" --embed-certs=true --kubeconfig="${instance}.kubeconfig"; kubectl config set-context default --cluster=kubernetes-the-hard-way --user=system:${instance} --kubeconfig=".\${instance}.kubeconfig"; kubectl config use-context default --kubeconfig="${instance}.kubeconfig"`
    * verify files generated
        * kube-proxy.kubeconfig
* generate kubeconfig file for kube-controller-manager service
    * `$INSTANCE="kube-controller-manager"; kubectl config set-cluster kubernetes-the-hard-way --certificate-authority=".\ca.pem" --embed-certs=true --server="https://${KUBERNETES_PUBLIC_ADDRESS}:6443" --kubeconfig="${instance}.kubeconfig"; kubectl config set-credentials system:${instance} --client-certificate="${instance}.pem" --client-key=".\${instance}-key.pem" --embed-certs=true --kubeconfig="${instance}.kubeconfig"; kubectl config set-context default --cluster=kubernetes-the-hard-way --user=system:${instance} --kubeconfig=".\${instance}.kubeconfig"; kubectl config use-context default --kubeconfig="${instance}.kubeconfig"`
    * verify files generated
        * kube-controller-manager.kubeconfig
* generate kubeconfig file for kube-scheduler service
    * `$INSTANCE="kube-scheduler"; kubectl config set-cluster kubernetes-the-hard-way --certificate-authority=".\ca.pem" --embed-certs=true --server="https://127.0.0.1:6443" --kubeconfig="${instance}.kubeconfig"; kubectl config set-credentials system:${instance} --client-certificate="${instance}.pem" --client-key=".\${instance}-key.pem" --embed-certs=true --kubeconfig="${instance}.kubeconfig"; kubectl config set-context default --cluster=kubernetes-the-hard-way --user=system:${instance} --kubeconfig=".\${instance}.kubeconfig"; kubectl config use-context default --kubeconfig="${instance}.kubeconfig"`
    * verify files generated
        * kube-scheduler.kubeconfig
* generate kubeconfig file for admin user
    * `$INSTANCE="admin"; kubectl config set-cluster kubernetes-the-hard-way --certificate-authority=".\ca.pem" --embed-certs=true --server="https://127.0.0.1:6443" --kubeconfig="${instance}.kubeconfig"; kubectl config set-credentials ${instance} --client-certificate="${instance}.pem" --client-key=".\${instance}-key.pem" --embed-certs=true --kubeconfig="${instance}.kubeconfig"; kubectl config set-context default --cluster=kubernetes-the-hard-way --user=${instance} --kubeconfig=".\${instance}.kubeconfig"; kubectl config use-context default --kubeconfig="${instance}.kubeconfig"`
    * verify files generated
        * kube-scheduler.kubeconfig
* distribute kubeconfig files 
    * workers get kubelet(worker) and kube-proxy kubeconfigs
    * worker-0
        * `$INSTANCE="worker-0"; gcloud compute scp "${instance}.kubeconfig" ${instance}:"${instance}.kubeconfig"; gcloud compute scp kube-proxy.kubeconfig ${instance}:kube-proxy.kubeconfig`
    * worker-1
        * `$INSTANCE="worker-1"; gcloud compute scp "${instance}.kubeconfig" ${instance}:"${instance}.kubeconfig"; gcloud compute scp kube-proxy.kubeconfig ${instance}:kube-proxy.kubeconfig`
    * worker-2
        * `$INSTANCE="worker-2"; gcloud compute scp "${instance}.kubeconfig" ${instance}:"${instance}.kubeconfig"; gcloud compute scp kube-proxy.kubeconfig ${instance}:kube-proxy.kubeconfig`
    * controllers get admin, kube-controller-manager and kube-scheduler kubeconfigs
    * controller-0
        * `$INSTANCE="controller-0"; gcloud compute scp admin.kubeconfig ${instance}:admin.kubeconfig; gcloud compute scp kube-controller-manager.kubeconfig ${instance}:kube-controller-manager.kubeconfig; gcloud compute scp kube-scheduler.kubeconfig ${instance}:kube-scheduler.kubeconfig`
    * controller-1
        * `$INSTANCE="controller-1"; gcloud compute scp admin.kubeconfig ${instance}:admin.kubeconfig; gcloud compute scp kube-controller-manager.kubeconfig ${instance}:kube-controller-manager.kubeconfig; gcloud compute scp kube-scheduler.kubeconfig ${instance}:kube-scheduler.kubeconfig`
    * controller-2
        * `$INSTANCE="controller-2"; gcloud compute scp admin.kubeconfig ${instance}:admin.kubeconfig; gcloud compute scp kube-controller-manager.kubeconfig ${instance}:kube-controller-manager.kubeconfig; gcloud compute scp kube-scheduler.kubeconfig ${instance}:kube-scheduler.kubeconfig`

