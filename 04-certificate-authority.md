* generate ca config file
    ```
    @'
    {
      "signing": {
        "default": {
          "expiry": "8760h"
        },
        "profiles": {
          "kubernetes": {
            "usages": ["signing", "key encipherment", "server auth", "client auth"],
            "expiry": "8760h"
          }
        }
      }
    }
    '@ | Out-File -encoding ASCII ca-config.json
    ```
* generate cert signing request
    ```
    @'
    {
      "CN": "Kubernetes",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Portland",
          "O": "Kubernetes",
          "OU": "CA",
          "ST": "Oregon"
        }
      ]
    }
    '@ | Out-File -encoding ASCII ca-csr.json
    ```
* generate certificate and private key
    * `cfssl gencert -initca .\ca-csr.json | cfssljson -bare ca`
* verify files generated
    * ca-key.pem
    * ca.csr
    * ca.pem
* generate admin cert signing request
    ```
    @'
    {
      "CN": "admin",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Portland",
          "O": "system:masters",
          "OU": "Kubernetes The Hard Way",
          "ST": "Oregon"
        }
      ]
    }
    '@ | Out-File -encoding ASCII admin-csr.json
    ```
* generate the admin client certificate
    * `cfssl gencert -ca="ca.pem" -ca-key="ca-key.pem" -config="ca-config.json" -profile=kubernetes .\admin-csr.json | cfssljson -bare admin`
    * verify files generated
        * admin-key.pem
        * admin.csr
        * admin.pem
* generate the worker certificates
    * worker-0
        * set vars
            * `$INSTANCE="worker-0"; $EXTERNAL_IP=$(gcloud compute instances describe ${instance} --format 'value(networkInterfaces[0].accessConfigs[0].natIP)'); $INTERNAL_IP=$(gcloud compute instances describe ${instance} --format 'value(networkInterfaces[0].networkIP)');`
        * create csr
            ```
            @'
            {
              "CN": "system:node:worker-0",
              "key": {
                "algo": "rsa",
                "size": 2048
              },
              "names": [
                {
                  "C": "US",
                  "L": "Portland",
                  "O": "system:nodes",
                  "OU": "Kubernetes The Hard Way",
                  "ST": "Oregon"
                }
              ]
            }
            '@ | Out-File -encoding ASCII ${instance}-csr.json
            ```
        * create cert
            * `cfssl gencert -ca="ca.pem" -ca-key="ca-key.pem" -config="ca-config.json" -hostname="${instance},${EXTERNAL_IP},${INTERNAL_IP}" -profile=kubernetes ${instance}-csr.json | cfssljson -bare ${instance}`
    * worker-1
        * set vars
            * `$INSTANCE="worker-1"; $EXTERNAL_IP=$(gcloud compute instances describe ${instance} --format 'value(networkInterfaces[0].accessConfigs[0].natIP)'); $INTERNAL_IP=$(gcloud compute instances describe ${instance} --format 'value(networkInterfaces[0].networkIP)');`
        * create csr
            ```
            @'
            {
              "CN": "system:node:worker-1",
              "key": {
                "algo": "rsa",
                "size": 2048
              },
              "names": [
                {
                  "C": "US",
                  "L": "Portland",
                  "O": "system:nodes",
                  "OU": "Kubernetes The Hard Way",
                  "ST": "Oregon"
                }
              ]
            }
            '@ | Out-File -encoding ASCII ${instance}-csr.json
            ```
        * create cert
            * `cfssl gencert -ca="ca.pem" -ca-key="ca-key.pem" -config="ca-config.json" -hostname="${instance},${EXTERNAL_IP},${INTERNAL_IP}" -profile=kubernetes ${instance}-csr.json | cfssljson -bare ${instance}`
    * worker-2
        * set vars
            * `$INSTANCE="worker-2"; $EXTERNAL_IP=$(gcloud compute instances describe ${instance} --format 'value(networkInterfaces[0].accessConfigs[0].natIP)'); $INTERNAL_IP=$(gcloud compute instances describe ${instance} --format 'value(networkInterfaces[0].networkIP)');`
        * create csr
            ```
            @'
            {
              "CN": "system:node:worker-2",
              "key": {
                "algo": "rsa",
                "size": 2048
              },
              "names": [
                {
                  "C": "US",
                  "L": "Portland",
                  "O": "system:nodes",
                  "OU": "Kubernetes The Hard Way",
                  "ST": "Oregon"
                }
              ]
            }
            '@ | Out-File -encoding ASCII ${instance}-csr.json
            ```
        * create cert
            * `cfssl gencert -ca="ca.pem" -ca-key="ca-key.pem" -config="ca-config.json" -hostname="${instance},${EXTERNAL_IP},${INTERNAL_IP}" -profile=kubernetes ${instance}-csr.json | cfssljson -bare ${instance}`
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
        * `$INSTANCE="worker-0"; gcloud compute scp .\ca.pem ${instance}:ca.pem; gcloud compute scp ".\${instance}-key.pem" ${instance}:"${instance}-key.pem"; gcloud compute scp ".\${instance}.pem" ${instance}:"${instance}.pem"`
    * worker-1
        * `$INSTANCE="worker-1"; gcloud compute scp .\ca.pem ${instance}:ca.pem; gcloud compute scp ".\${instance}-key.pem" ${instance}:"${instance}-key.pem"; gcloud compute scp ".\${instance}.pem" ${instance}:"${instance}.pem"`
    * worker-2
        * `$INSTANCE="worker-2"; gcloud compute scp .\ca.pem ${instance}:ca.pem; gcloud compute scp ".\${instance}-key.pem" ${instance}:"${instance}-key.pem"; gcloud compute scp ".\${instance}.pem" ${instance}:"${instance}.pem"`
    * controller-0
        * `$INSTANCE="controller-0"; gcloud compute scp .\ca.pem ${instance}:ca.pem; gcloud compute scp ".\ca-key.pem" ${instance}:ca-key.pem; gcloud compute scp ".\kubernetes-key.pem" ${instance}:kubernetes-key.pem; gcloud compute scp ".\kubernetes.pem" ${instance}:kubernetes.pem; gcloud compute scp ".\service-account-key.pem" ${instance}:service-account-key.pem; gcloud compute scp ".\service-account.pem" ${instance}:service-account.pem`
    * controller-1
        * `$INSTANCE="controller-1"; gcloud compute scp .\ca.pem ${instance}:ca.pem; gcloud compute scp ".\ca-key.pem" ${instance}:ca-key.pem; gcloud compute scp ".\kubernetes-key.pem" ${instance}:kubernetes-key.pem; gcloud compute scp ".\kubernetes.pem" ${instance}:kubernetes.pem; gcloud compute scp ".\service-account-key.pem" ${instance}:service-account-key.pem; gcloud compute scp ".\service-account.pem" ${instance}:service-account.pem`
    * controller-2
        * `$INSTANCE="controller-2"; gcloud compute scp .\ca.pem ${instance}:ca.pem; gcloud compute scp ".\ca-key.pem" ${instance}:ca-key.pem; gcloud compute scp ".\kubernetes-key.pem" ${instance}:kubernetes-key.pem; gcloud compute scp ".\kubernetes.pem" ${instance}:kubernetes.pem; gcloud compute scp ".\service-account-key.pem" ${instance}:service-account-key.pem; gcloud compute scp ".\service-account.pem" ${instance}:service-account.pem`
