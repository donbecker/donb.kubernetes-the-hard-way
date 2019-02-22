* all commands below must be run on all controllers
* create kubernetes config directory
    * `sudo mkdir -p /etc/kubernetes/config`
* download k8s
```
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kubectl"
```
* install k8s
    * `chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl`
    * `sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/`
* config k8s api server
    * `sudo mkdir -p /var/lib/kubernetes/`
    * `sudo mv ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem service-account-key.pem service-account.pem encryption-config.yaml /var/lib/kubernetes/`
* retrieve internal IP of instance
    * `INTERNAL_IP=$(curl -s -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/ip)`
* create kube-apiserver.service systemd unit file
```
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=Initializers,NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --enable-swagger-ui=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\
  --etcd-servers=https://10.240.0.10:2379,https://10.240.0.11:2379,https://10.240.0.12:2379 \\
  --event-ttl=1h \\
  --experimental-encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\
  --kubelet-https=true \\
  --runtime-config=api/all \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
* move kube-controller-manager kubeconfig to correct place
    * `sudo mv kube-controller-manager.kubeconfig /var/lib/kubernetes/`
* create kube-controller-manager.service systemd unit file
```
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --address=0.0.0.0 \\
  --cluster-cidr=10.200.0.0/16 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
* move kube-scheduler kubeconfig to correct place
    * `sudo mv kube-scheduler.kubeconfig /var/lib/kubernetes/`
* create the kube-scheduler.yaml config file
```
cat <<EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: componentconfig/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
```
* create the kube-scheduler.service systemd unit file
```
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
* start the controller services
    * `sudo systemctl daemon-reload`
    * `sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler`
    * `sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler`
* install nginx to handle health checks
    * `sudo apt-get install -y nginx`
* create nginx config file
```
cat > kubernetes.default.svc.cluster.local <<EOF
server {
  listen      80;
  server_name kubernetes.default.svc.cluster.local;

  location /healthz {
     proxy_pass                    https://127.0.0.1:6443/healthz;
     proxy_ssl_trusted_certificate /var/lib/kubernetes/ca.pem;
  }
}
EOF
```
* move nginx config file to correct directory
    * `sudo mv kubernetes.default.svc.cluster.local /etc/nginx/sites-available/kubernetes.default.svc.cluster.local`
* create symlink to enable nginx
    * `sudo ln -s /etc/nginx/sites-available/kubernetes.default.svc.cluster.local /etc/nginx/sites-enabled/`
* start nginx
    * `sudo systemctl restart nginx`
    * `sudo systemctl enable nginx`
* verify cluster (run on all controllers)
    * `kubectl get componentstatuses --kubeconfig admin.kubeconfig`
* test nginx HTTP health check proxy (run on all controllers
    * `curl -H "Host: kubernetes.default.svc.cluster.local" -i http://127.0.0.1/healthz`
* from the controller-0 server:
    * create the system:kube-apiserver-to-kubelet cluster role
      ```
      cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
      apiVersion: rbac.authorization.k8s.io/v1beta1
      kind: ClusterRole
      metadata:
        annotations:
          rbac.authorization.kubernetes.io/autoupdate: "true"
        labels:
          kubernetes.io/bootstrapping: rbac-defaults
        name: system:kube-apiserver-to-kubelet
      rules:
        - apiGroups:
            - ""
          resources:
            - nodes/proxy
            - nodes/stats
            - nodes/log
            - nodes/spec
            - nodes/metrics
          verbs:
            - "*"
      EOF
      ```
   * verify
      * `kubectl get clusterroles system:kube-apiserver-to-kubelet`
   * bind the system:kube-apiserver-to-kubelet cluster role to the kubernetes user
   ```
   cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
   apiVersion: rbac.authorization.k8s.io/v1beta1
   kind: ClusterRoleBinding
   metadata:
     name: system:kube-apiserver
     namespace: ""
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: ClusterRole
     name: system:kube-apiserver-to-kubelet
   subjects:
     - apiGroup: rbac.authorization.k8s.io
       kind: User
       name: kubernetes
   EOF
   ```
   * verify
       * `kubectl get clusterrolebinding system:kube-apiserver`
* from the workstation (not the controller instances) create the GCP resources for the load balancer
    * retrieve public IP address
        * `$KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way --region $(gcloud config get-value compute/region) --format 'value(address)')`
    * create health check
        * `gcloud compute http-health-checks create kubernetes --description "Kubernetes Health Check" --host "kubernetes.default.svc.cluster.local" --request-path "/healthz"`
    * create firewall rules for health check
        * `gcloud compute firewall-rules create kubernetes-the-hard-way-allow-health-check --network kubernetes-the-hard-way --source-ranges 209.85.152.0/22,209.85.204.0/22,35.191.0.0/16 --allow tcp`
    * create target pool
        * `gcloud compute target-pools create kubernetes-target-pool --http-health-check kubernetes`
    * add controllers to target pool
        * `gcloud compute target-pools add-instances kubernetes-target-pool --instances controller-0,controller-1,controller-2`
    * create forwarding rule
        * `gcloud compute forwarding-rules create kubernetes-forwarding-rule --address ${KUBERNETES_PUBLIC_ADDRESS} --ports 6443 --region $(gcloud config get-value compute/region) --target-pool kubernetes-target-pool`
    * install curl via Chocolatey
        * `cinst -y curl`
    * make http request for the k8s cluster version info
        * `Remove-Item Alias:curl; curl --cacert ca.pem https://${KUBERNETES_PUBLIC_ADDRESS}:6443/version`
* next: https://github.com/donbecker/donb.kubernetes-the-hard-way/blob/master/09-bootstrapping-kubernetes-workers.md
