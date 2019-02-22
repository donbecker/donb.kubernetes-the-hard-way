* connect to the first controller and create the encryption config file
    * `Start-Process -FilePath "gcloud" -ArgumentList "compute ssh controller-0"`
* paste into shell on controller
   * generate encryption key
       * `ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)`
   * create the encryption file
   ```
   cat > encryption-config.yaml <<EOF
   kind: EncryptionConfig
   apiVersion: v1
   resources:
     - resources:
         - secrets
       providers:
         - aescbc:
             keys:
               - name: key1
                 secret: ${ENCRYPTION_KEY}
         - identity: {}
   EOF
   ```
* close shell on controller
* copy the encryption config file from the controller to local
    * `$INSTANCE="controller-0"; gcloud compute scp ${instance}:encryption-config.yaml encryption-config.yaml`
* distribute encryption config file to the other controllers
    * controller-1
        * `$INSTANCE="controller-1"; gcloud compute scp encryption-config.yaml ${instance}:encryption-config.yaml`
    * controller-2
        * `$INSTANCE="controller-2"; gcloud compute scp encryption-config.yaml ${instance}:encryption-config.yaml`
* next: https://github.com/donbecker/donb.kubernetes-the-hard-way/blob/master/07-bootstrapping-etcd.md
