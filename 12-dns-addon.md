* from workstation, install coredns cluster addon
    * `kubectl apply -f https://storage.googleapis.com/kubernetes-the-hard-way/coredns.yaml`
* list pods create by kube-dns deployment
    * `kubectl get pods -l k8s-app=kube-dns -n kube-system`
* verify dns working via busybox deploy
    * deploy busybox
        * `kubectl run busybox --image=busybox:1.28 --command -- sleep 3600`
    * list busybox pod
        * `kubectl get pods -l run=busybox`
    * grab pod name of busybox deploy
        * `$POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")`
    * run DNS lookup for the kubernetes service from the busybox pod
        * `kubectl exec -ti $POD_NAME -- nslookup kubernetes`
* next: https://github.com/donbecker/donb.kubernetes-the-hard-way/blob/master/13-smoke-test.md
