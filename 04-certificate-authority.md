* generate ca config file, certificate and private key
    * `cfssl gencert -initca .\ca-csr.json | cfssljson -bare ca`
* verify files generated
    * ca-key.pem
    * ca.csr
    * ca.pem
* generate the admin client certificate
    * `cfssl gencert -ca="ca.pem" -ca-key="ca-key.pem" -config="ca-config.json" -profile=kubernetes .\admin-csr.json | cfssljson -bare admin`
    * verify files generated
        * admin-key.pem
        * admin.csr
        * admin.pem
* generate the worker certificates
    * worker-0
        * `$INSTANCE="worker-0"; $EXTERNAL_IP=$(gcloud compute instances describe ${instance} --format 'value(networkInterfaces[0].accessConfigs[0].natIP)'); $INTERNAL_IP=$(gcloud compute instances describe ${instance} --format 'value(networkInterfaces[0].networkIP)'); cfssl gencert -ca="ca.pem" -ca-key="ca-key.pem" -config="ca-config.json" -hostname="${instance},${EXTERNAL_IP},${INTERNAL_IP}" -profile=kubernetes ${instance}-csr.json | cfssljson -bare ${instance}`
    * worker-1
        * `$INSTANCE="worker-1"; $EXTERNAL_IP=$(gcloud compute instances describe ${instance} --format 'value(networkInterfaces[0].accessConfigs[0].natIP)'); $INTERNAL_IP=$(gcloud compute instances describe ${instance} --format 'value(networkInterfaces[0].networkIP)'); cfssl gencert -ca="ca.pem" -ca-key="ca-key.pem" -config="ca-config.json" -hostname="${instance},${EXTERNAL_IP},${INTERNAL_IP}" -profile=kubernetes ${instance}-csr.json | cfssljson -bare ${instance}`
    * worker-2
        * `$INSTANCE="worker-2"; $EXTERNAL_IP=$(gcloud compute instances describe ${instance} --format 'value(networkInterfaces[0].accessConfigs[0].natIP)'); $INTERNAL_IP=$(gcloud compute instances describe ${instance} --format 'value(networkInterfaces[0].networkIP)'); cfssl gencert -ca="ca.pem" -ca-key="ca-key.pem" -config="ca-config.json" -hostname="${instance},${EXTERNAL_IP},${INTERNAL_IP}" -profile=kubernetes ${instance}-csr.json | cfssljson -bare ${instance}`
    * verify files generated
        * worker-0-key.pem
        * worker-0.csr
        * worker-0.pem
        * worker-1-key.pem
        * worker-1.csr
        * worker-1.pem
        * worker-2-key.pem
        * worker-2.csr
        * worker-2.pem
* generate the kube-controller-manager client certificate and key
    * `cfssl gencert -ca="ca.pem" -ca-key="ca-key.pem" -config="ca-config.json" -profile=kubernetes .\kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager`
    * verify files generated
        * kube-controller-manager-key.pem
        * kube-controller-manager.csr
        * kube-controller-manager.pem
* generate the kube-proxy client certificate and key
    * `cfssl gencert -ca="ca.pem" -ca-key="ca-key.pem" -config="ca-config.json" -profile=kubernetes .\kube-proxy-csr.json | cfssljson -bare kube-proxy`
    * verify files generated
        * kube-proxy-key.pem
        * kube-proxy.csr
        * kube-proxy.pem
* generate the kube-scheduler client certificate and key
    * `cfssl gencert -ca="ca.pem" -ca-key="ca-key.pem" -config="ca-config.json" -profile=kubernetes .\kube-scheduler-csr.json | cfssljson -bare kube-scheduler`
    * verify files generated
        * kube-scheduler-key.pem
        * kube-scheduler.csr
        * kube-scheduler.pem
* generate kubernetes api server certificate and key, adding our public IP to the list of subject alternative names
    * `$KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way --region $(gcloud config get-value compute/region) --format 'value(address)')`
    * `cfssl gencert -ca="ca.pem" -ca-key="ca-key.pem" -config="ca-config.json" -hostname="10.32.0.1,10.240.0.10,10.240.0.11,10.240.0.12,${KUBERNETES_PUBLIC_ADDRESS},127.0.0.1,kubernetes.default" -profile=kubernetes .\kubernetes-csr.json | cfssljson -bare kubernetes`
    * verify files generated
        * kubernetes-key.pem
        * kubernetes.csr
        * kubernetes.pem
* generate the service-account client certificate and key
    * `cfssl gencert -ca="ca.pem" -ca-key="ca-key.pem" -config="ca-config.json" -profile=kubernetes .\service-account-csr.json | cfssljson -bare service-account`
    * verify files generated
        * service-account-key.pem
        * service-account.csr
        * service-account.pem
* distribute the client and server certificates
    * worker-0
        * `$INSTANCE="worker-0"; gcloud compute scp .\ca.pem ${instance}:~; gcloud compute scp ".\${instance}-key.pem" ${instance}:~; gcloud compute scp ".\${instance}.pem" ${instance}:~`
    * worker-1
        * `$INSTANCE="worker-1"; gcloud compute scp .\ca.pem ${instance}:~; gcloud compute scp ".\${instance}-key.pem" ${instance}:~; gcloud compute scp ".\${instance}.pem" ${instance}:~`
    * worker-2
        * `$INSTANCE="worker-2"; gcloud compute scp .\ca.pem ${instance}:~; gcloud compute scp ".\${instance}-key.pem" ${instance}:~; gcloud compute scp ".\${instance}.pem" ${instance}:~`
    * controller-0
        * `$INSTANCE="controller-0"; gcloud compute scp .\ca.pem ${instance}:~; gcloud compute scp ".\ca-key.pem" ${instance}:~; gcloud compute scp ".\kubernetes-key.pem" ${instance}:~; gcloud compute scp ".\kubernetes.pem" ${instance}:~; gcloud compute scp ".\service-account-key.pem" ${instance}:~; gcloud compute scp ".\service-account.pem" ${instance}:~`
    * controller-1
        * `$INSTANCE="controller-1"; gcloud compute scp .\ca.pem ${instance}:~; gcloud compute scp ".\ca-key.pem" ${instance}:~; gcloud compute scp ".\kubernetes-key.pem" ${instance}:~; gcloud compute scp ".\kubernetes.pem" ${instance}:~; gcloud compute scp ".\service-account-key.pem" ${instance}:~; gcloud compute scp ".\service-account.pem" ${instance}:~`
    * controller-2
        * `$INSTANCE="controller-2"; gcloud compute scp .\ca.pem ${instance}:~; gcloud compute scp ".\ca-key.pem" ${instance}:~; gcloud compute scp ".\kubernetes-key.pem" ${instance}:~; gcloud compute scp ".\kubernetes.pem" ${instance}:~; gcloud compute scp ".\service-account-key.pem" ${instance}:~; gcloud compute scp ".\service-account.pem" ${instance}:~`