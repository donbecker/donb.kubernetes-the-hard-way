* create a generic secret
    * `kubectl create secret generic kubernetes-the-hard-way --from-literal="mykey=mydata"`
* print hexdump of the secret
    * `gcloud compute ssh controller-0 --command "sudo ETCDCTL_API=3 etcdctl get --endpoints=https://127.0.0.1:2379 --cacert=/etc/etcd/ca.pem --cert=/etc/etcd/kubernetes.pem  --key=/etc/etcd/kubernetes-key.pem /registry/secrets/default/kubernetes-the-hard-way" | Format-Hex`
* verify the output is prefixed with "k8s:enc:aescbc:v1:key1" (aescbc provider, key1 encryption key)
* create a deployment for nginx
    * `kubectl run nginx --image=nginx`
* list the nginx pod 
    * `kubectl get pods -l run=nginx`
* grab pod name of nginx 
    * `$POD_NAME=$(kubectl get pods -l run=nginx -o jsonpath="{.items[0].metadata.name}")`
* port forward 8080 locally to 80 of the nginx pod
    * `kubectl port-forward $POD_NAME 8080:80`
* open new powershell window, and use curl to verify the nginx server
    * `Remove-Item Alias:curl; curl --head http://127.0.0.1:8080`
    * verify
```
HTTP/1.1 200 OK
  0   612    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
Server: nginx/1.15.5
Date: Sun, 07 Oct 2018 20:13:45 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 02 Oct 2018 14:49:27 GMT
Connection: keep-alive
ETag: "5bb38577-264"
Accept-Ranges: bytes
```
* cancel the port forward 
    * `Ctrl+C`
* pull nginx pod logs
    * `kubectl logs $POD_NAME`
* validating executing a command in a pod via kubectl by printing the nginx version
    * `kubectl exec -ti $POD_NAME -- nginx -v`
* expose the nginx deployment using a nodeport service
    * `kubectl expose deployment nginx --port 80 --type NodePort`
* retrieve the node port assigned to the nginx nodeport service
    * `$NODE_PORT=$(kubectl get svc nginx --output=jsonpath='{range .spec.ports[0]}{.nodePort}')`
* create firewall rule to allow remote access to the port
    * `gcloud compute firewall-rules create kubernetes-the-hard-way-allow-nginx-service --allow=tcp:${NODE_PORT} --network kubernetes-the-hard-way`
* retrieve the external IP of one of the worker instances
    * `$EXTERNAL_IP=$(gcloud compute instances describe worker-0 --format 'value(networkInterfaces[0].accessConfigs[0].natIP)')`
* make an HTTP request using the IP and the node port
    * `Remove-Item Alias:curl; curl -I http://${EXTERNAL_IP}:${NODE_PORT}`
* verify curl output from nginx
```
HTTP/1.1 200 OK
Server: nginx/1.15.5
Date: Sun, 07 Oct 2018 20:37:19 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 02 Oct 2018 14:49:27 GMT
Connection: keep-alive
ETag: "5bb38577-264"
Accept-Ranges: bytes
```
* create untrusted workload
```
@'
apiVersion: v1
kind: Pod
metadata:
  name: untrusted
  annotations:
    io.kubernetes.cri.untrusted-workload: "true"
spec:
  containers:
    - name: webserver
      image: gcr.io/hightowerlabs/helloworld:2.0.0
'@ | Out-File -encoding ASCII untrusted.yaml
```
* deploy the untrusted workload 
    * `kubectl apply -f .\untrusted.yaml`
* verify pods running 
    * `kubectl get pods -o wide`
* get node name where untrusted pod is running 
    * `$INSTANCE_NAME=$(kubectl get pod untrusted --output=jsonpath='{.spec.nodeName}')`
* connect to the instance
    * `Start-Process -FilePath "gcloud" -ArgumentList "compute ssh $INSTANCE_NAME"`
* on the instance (via ssh)
    * list containers running under gVisor
        * `sudo runsc --root  /run/containerd/runsc/k8s.io list`
    * get ID of untrusted pod
        * `POD_ID=$(sudo crictl -r unix:///var/run/containerd/containerd.sock pods --name untrusted -q)`
    * get ID of webserver container running in the untrusted pod
        * `CONTAINER_ID=$(sudo crictl -r unix:///var/run/containerd/containerd.sock ps -p ${POD_ID} -q)`
    * list processes running in webserver container
        * `sudo runsc --root /run/containerd/runsc/k8s.io ps ${CONTAINER_ID}`
