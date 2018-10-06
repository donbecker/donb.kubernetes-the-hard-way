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