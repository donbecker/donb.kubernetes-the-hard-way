* Install SSL tools
    * make tools folder: c:\tools\cfssl\
    * cfssl: https://pkg.cfssl.org/R1.2/cfssl_windows-amd64.exe
        * move to tools folder, rename to cfssl.exe, unblock
    * cfssljson: https://pkg.cfssl.org/R1.2/cfssljson_windows-amd64.exe
        * move to tools folder, rename to cfssljson.exe, unblock
    * add to SYSTEM env var path
    * verify
        * `cfssl version`
* Install kubectl
    * `cinst -y kubernetes-cli`
    * verify
        * `kubectl version --client`


