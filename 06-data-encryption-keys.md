* generate encryption config file
    * `$ENCRYPTION_KEY=[System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes([guid]::NewGuid())); "kind: EncryptionConfig`r`napiVersion: v1`r`nresources:`r`n  - resources:`r`n      - secrets`r`n    providers:`r`n      - aescbc:`r`n          keys:`r`n            - name: key1`r`n              secret: ${ENCRYPTION_KEY}`r`n      - identity: {}" | Out-File encryption-config.yaml -Encoding ascii`
* fix line endings on encryption config file: change from "CRLF" to "LF" in VSCode and save
* distribute encryption config file to each controller
    * controller-0
        * `$INSTANCE="controller-0"; gcloud compute scp encryption-config.yaml ${instance}:encryption-config.yaml`
    * controller-1
        * `$INSTANCE="controller-1"; gcloud compute scp encryption-config.yaml ${instance}:encryption-config.yaml`
    * controller-2
        * `$INSTANCE="controller-2"; gcloud compute scp encryption-config.yaml ${instance}:encryption-config.yaml`