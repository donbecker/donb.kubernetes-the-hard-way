* ssh into all controllers
    * `Start-Process -FilePath "gcloud" -ArgumentList "compute ssh controller-0"`
    * `Start-Process -FilePath "gcloud" -ArgumentList "compute ssh controller-1"`
    * `Start-Process -FilePath "gcloud" -ArgumentList "compute ssh controller-2"`
* on each controller, run the following commands
* download etcd
    * `wget -q --show-progress --https-only --timestamping "https://github.com/coreos/etcd/releases/download/v3.3.9/etcd-v3.3.9-linux-amd64.tar.gz"`
* unzip and install
    * `tar -xvf etcd-v3.3.9-linux-amd64.tar.gz`
    * `sudo mv etcd-v3.3.9-linux-amd64/etcd* /usr/local/bin/`
* configure etcd
    * `sudo mkdir -p /etc/etcd /var/lib/etcd`
    * `sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/`
* retrieve instance private IP
    * `INTERNAL_IP=$(curl -s -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/ip)`
* set etcd name to hostname
    * `ETCD_NAME=$(hostname -s)`
* create the etcd.service yaml file
```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster controller-0=https://10.240.0.10:2380,controller-1=https://10.240.0.11:2380,controller-2=https://10.240.0.12:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
* start etcd
    * `sudo systemctl daemon-reload`
    * `sudo systemctl enable etcd`
    * `sudo systemctl start etcd`
* list cluster members
```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```  
* next: https://github.com/donbecker/donb.kubernetes-the-hard-way/blob/master/08-bootstrapping-kubernetes-controllers.md
