* generate encryption config file
    * `$ENCRYPTION_KEY=$(-join ((48..57) * 32 | Get-Random -Count 32 | % {[char]$_})); "kind: EncryptionConfig`r`napiVersion: v1`r`nresources:`r`n  - resources:`r`n      - secrets`r`n    providers:`r`n      - aescbc:`r`n          keys:`r`n            - name: key1`r`n              secret: ${ENCRYPTION_KEY}`r`n      - identity: {}" > encryption-config.yaml`
* distribute encryption config file to each controller
    * controller-0
        * `$INSTANCE="controller-0"; gcloud compute scp encryption-config.yaml ${instance}:~`
    * controller-1
        * `$INSTANCE="controller-1"; gcloud compute scp encryption-config.yaml ${instance}:~`
    * controller-2
        * `$INSTANCE="controller-2"; gcloud compute scp encryption-config.yaml ${instance}:~`