# Exercise 11 (Cluster key infrastructure)

## Preface:
#### If you haven't looked yet, or just don't know. You may have already asked yourself. How are my commands authenticated to k8s to allow me access? When you setup minikube, the setup process created a credential for you to access the cluster. It's stored in a common location ``` ~/.kube/config ``` lets have a look.

#### Open up a terminal and print out the contents of this file.
```bash
# cat ~/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /root/.minikube/ca.crt
    server: https://<IP_OF_YOUR_MINIKUBE>:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /root/.minikube/client.crt
    client-key: /root/.minikube/client.key
```
#### A few things to point out here, see all the .crt and .key files referenced here. It's because kubernetes is using PKI at the heart of all authentication. Minikube as part of it's setup make this easy and there are lots of other open source projects that do the same process of "standing up" a kubernetes cluster easy. it wasn't always this way. In the past if you wanted to setup a cluster you had to initialize each component separately. etcd and api server together as CA and secret storage, then other segments like scheduler and controller-manager etc. could be setup after because X.509 certificates could be signed by the api server once it was up. 

All k8s components, (scheduler, controller-manager, kube-proxy, etc.) they all use mutual authentication to communicate with each other. For example, lets say you have a second Node (computer) for which you want to run workloads, you would need some of the CA cert information, which you could get with a command like this:
```bash
# openssl x509 -pubkey -in ~/.minikube/ca.crt \
	| openssl rsa -pubin -outform der 2>/dev/null \
	| openssl dgst -sha256 -hex \
	| sed 's/^.* //'

ea582048c5777ec70eea2ee3daf06a461b0e352ef4365c7cb024f9009359197e
```
As well as a token, ``` kubeadm token list ``` Sadly minikube was never designed to be a multi-node environment setup. (planed for the future) Howesr we will use a diferent tool ```k3s``` which we will use later to experiment with this. 
The point is, even between node controllers (kubelet) running on other nodes, all communication is mTLS or mutual X.509 authenticated.

### From a workload traffic perspective, there is lots we can do to experiment with PKI. But first lets grab some useful tools, please note that all of the private key creation, and CSR generation stuff can be done via openssl, I'm just using cfssl because it will tie into other exercises later. 

```bash
curl -LO https://pkg.cfssl.org/R1.2/cfssl-bundle_linux-amd64
curl -LO https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
curl -LO https://pkg.cfssl.org/R1.2/cfssl-newkey_linux-amd64
curl -LO https://pkg.cfssl.org/R1.2/cfssl-scan_linux-amd64
curl -LO https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
curl -LO https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
chmod +x cfssl*
mv cfssl-bundle_linux-amd64 cfssl-bundle
mv cfssl-certinfo_linux-amd64 cfssl-certinfo
mv cfssl-newkey_linux-amd64 cfssl-newkey
mv cfssl-scan_linux-amd64 cfssl-scan
mv cfssl_linux-amd64 cfssl
mv cfssljson_linux-amd64 cfssljson
mv -fv cfssl* /usr/bin/
```
Now you should have ___"cfssl"___ installed on your kali system, please [download](https://pkg.cfssl.org/) other binaries if you are fallowing this via a different system. (macOS users can just run ```brew install cfssl cfssljson```) Next we need to get a listing of our current wordpress pods and services to put into the request.
```bash
# kubectl get pods,svc -o wide
NAME                             READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
pod/mysql-6c8b769d74-sd5f6       1/1     Running   0          19h   172.17.0.5   minikube   <none>           <none>
pod/wordpress-6d949c5b75-6bk2h   1/1     Running   0          19h   172.17.0.6   minikube   <none>           <none>
pod/wordpress-6d949c5b75-zk7tm   1/1     Running   0          18h   172.17.0.7   minikube   <none>           <none>

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE   SELECTOR
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP    19h   <none>
service/mysql-svc    ClusterIP   10.96.17.119   <none>        3306/TCP   19h   app=mysql
service/wp-svc       ClusterIP   10.96.91.157   <none>        80/TCP     18h   app=wordpress
```
Now we will need to grab some of the IP and name information for our request. (we are running this exercise from within the ```default``` namespace)
```bash
# cat <<EOF | cfssl genkey - | cfssljson -bare server
{
  "hosts": [
    "wp-svc.default.svc.cluster.local",
    "wordpress-6d949c5b75-6bk2h.default.pod.cluster.local",
    "wordpress-6d949c5b75-zk7tm.default.pod.cluster.local",
    "10.96.91.157",
    "172.17.0.6",
    "172.17.0.7"
  ],
  "CN": "wp-svc.default.pod.cluster.local",
  "key": {
    "algo": "ecdsa",
    "size": 256
  }
}
EOF
```
As you can see my pod names and IP's will be different then yours, you may want to copy the command above to a text editor or notepad to edit BEFORE you execute it! obviously substituting your pod names and service IP's etc.

The above command will create two files in whatever directory you are currently in. ```server-key.pem``` and ```server.csr``` now we will request it to be signed by the native k8s CA with this command.
```bash
# cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: wp-svc.default
spec:
  request: $(cat server.csr | base64 | tr -d '\n')
  usages:
  - digital signature
  - key encipherment
  - server auth
EOF
```
Please verify it's been received via the command:
```bash
# kubectl get csr
```
Because you are the admin of your cluster, you are able to approve them:
```bash
# kubectl certificate approve wp-svc.default
```
Then do another ```kubectl get csr```
```bash
# kubectl get csr
NAME             AGE   REQUESTOR              CONDITION
csr-j6sgr        42m   system:node:minikube   Approved,Issued
wp-svc.default   37s   minikube-user          Approved,Issued
```
It should say Approved now. And you should now be able to download the signed cert and writ it to a file. We can do that via ```kubectl``` and "print out" the certificate status via the jasonpath, like this:
```bash
# kubectl get csr wp-svc.default -o jsonpath='{.status.certificate}' \
    | base64 --decode > server.crt
```
You should now have a ___"server.crt"___ file in your local dir, like this. (you can ```cat``` the contents and see a Base64 encoded PEM format cert)
```bash
# ls
server-key.pem	server.crt	server.csr
# cat server.crt
-----BEGIN CERTIFICATE-----
MIIC5TCCAc2gAwIBAgIRAO2QNT1Bu3aLKJmTJEE06tkwDQYJKoZIhvcNAQELBQAw
FTETMBEGA1UEAxMKbWluaWt1YmVDQTAeFw0yMDAzMDgyMzUxMDJaFw0yMTAzMDgy
MzUxMDJaMCsxKTAnBgNVBAMTIHdwLXN2Yy5kZWZhdWx0LnBvZC5jbHVzdGVyLmxv
Y2FsMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEbh0Up4r/IYnIC0tAN+d7nHbr
......
```
With these three files you can setup tls on a service, or more importantly an ingress service! (we will get into ingress soon)

## Review:
### As you can see, we can leverage the ___"in built"___ [ca](https://en.wikipedia.org/wiki/Certificate_authority) created within k8s to sign certificates needed for workload traffic. In addition to manually provisioning keys and certs (as well as signing them) there are many Automated projects out in the open source ecosystem. Projects like [cert-manager](https://github.com/jetstack/cert-manager) managed by the JetStack group are excellent "drop in" tools to help manage and automate signing and renewal of expiring certs. 

## Clean op:
#### Please keep the wordpress pods and service around, we will use them soon. In addition save the ___"service.crt"___ and ___"server-key.pem"___ files as we will use them later as well.

[Return to schedule](../../Docs/SCHEDULE.md)