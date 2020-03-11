# Exercise 12 (Theres a better way)

## Note:
### Altho we will go through a typical setup for demonstration purposes, it is highly recommended to setup [Vault](https://www.vaultproject.io/) with redudency, multiple key custodians and with cert-manager bound to AppRole! (All of which are lengthy and beyond the time alotment for the trining, however I highly encourage you to investagate there extended setups if you wish to build a production setup! You have been warned ...)

## Preface:
Being able to handle certificate signing all within k8s is great, but there is a better way. One which takes advantage of kubernetes automation, scheduling and declarative configuration nature. I speak of course, of the [cert-manager](https://cert-manager.io/docs/) project. As an extendable framework to handle private key creation, certificate signing requests and the backend Issuer integration; 

## Prep:
Navigate to the ``` KernelCon2020K8s/Exercises/Defend/Files/defend_e12 ``` folder and do an ```ls``` to ensure you can see / access the correct files for this exercise.
```bash
# ls
cert-manager-policy.hcl	docker-compose.yaml
```

## Vault Setup

Download and install vault 
```bash
# curl -LO https://releases.hashicorp.com/vault/1.3.2/vault_1.3.2_linux_amd64.zip
# unzip vault_*.zip
# chmod +x vault
# cp -fv vault /usr/bin/
```
Ensure that vault is in your path and executable:
```bash
# vault version
Vault v1.3.2
```
Use docker-compose to ___"spin-up"___ an instance of vault server:
```bash
# export VAULT_ADDR='http://127.0.0.1:8200'
# export vault_root_token=$(openssl rand -base64 14)
# docker-compose -f docker-compose.yaml up -d
# unset vault_root_token
```
Then we need to grab two values from the log output of the vault startup.
```bash
# docker-compose -f docker-compose.yaml logs |grep -e "\(Unseal Key\|Root Token\)"
vault_1  | Unseal Key: 5OI8ylZj/ZMEHCeGJCrGmZMb3t7JtbLXCxzyDjxsU7w=
vault_1  | Root Token: zg0NcLljIC25rMnqPZY=
```
Your values will be random and unique, copy them and save them for later. (like in notepad etc.)

Now lets check to ensure that the vault server is available:
```bash
# curl http://127.0.0.1:8200/v1/sys/seal-status |jq

{
  "type": "shamir",
  "initialized": true,
  "sealed": false,
  "t": 1,
  "n": 1,
  "progress": 0,
  "nonce": "",
  "version": "1.3.3",
  "migration": false,
  "cluster_name": "vault-cluster-add47858",
  "cluster_id": "9b1ad728-d89d-32a3-ec93-cc79de785e6c",
  "recovery_seal": false,
  "storage_type": "inmem"
}
```
Now we should grab our ___"vault token"___ value from the log output so we can use it in the future, for now just store it in an ENV VAR.
```bash
export VAULT_TOKEN=$(docker-compose -f docker-compose.yaml logs |grep -e "Root Token" |cut -d ":" -f 2 |tr -d " ")
```
Now that we know our own ___"vault"___ server is online and available, we will need to setup ___"pki"___ . Because we have set ___VAULT_TOKEN___ and ___VAULT_ADDR___ all of our calls to vault will be authenticated automatically during our setup.

### PKI Setup
![Vault PKI](Files/images/vault-pki-4.png)
#### We will be setting up a 3 tier PKI infrastructure, Root CA, Intermediate CA and "cert-manager".

Enable pki via vault cli
```bash
vault secrets enable pki
```
Set the max "time to live" of the CA to 10 years
```bash
vault secrets tune -max-lease-ttl=8760h pki
```
Generate the root certificate
```bash
vault write -field=certificate pki/root/generate/internal \
        common_name="kernelcon2020k8s.org" \
        ttl=87600h > CA_cert.crt
```
Configure the CA and CRL URLs. (This is required)
```bash
vault write pki/config/urls \
        issuing_certificates="http://127.0.0.1:8200/v1/pki/ca" \
        crl_distribution_points="http://127.0.0.1:8200/v1/pki/crl"
```
Create a role for the CA
```bash
vault write pki/roles/kernelcon2020k8s.org \
    allowed_domains=kernelcon2020k8s.org,cluster.local,svc,pod \
    allow_subdomains=true \
    max_ttl=72h
```
Enable pki_int path for intermediate CA
```bash
vault secrets enable -path=pki_int pki
```
Tune the int CA TTL similar to the Root CA (7 years)
```bash
vault secrets tune -max-lease-ttl=61320h pki_int
```
Now generate our intermediate certificate signing request ...
```bash
vault write -format=json pki_int/intermediate/generate/internal \
        common_name="kernelcon2020k8s.org Intermediate Authority" \
        | jq -r '.data.csr' > pki_intermediate.csr
```
Sign it with the CA
```bash
vault write -format=json pki/root/sign-intermediate csr=@pki_intermediate.csr \
        format=pem_bundle ttl="43800h" \
        | jq -r '.data.certificate' > intermediate.cert.pem
```
Sweet, we have our PKI setup, now we just need to setup permissions to access it.
Create a user role for the "cert-manager" to use when talking to vault:
```bash
vault write pki_int/roles/cert-manager \
	allowed_domains=kernelcon2020k8s.org,cluster.local,svc,pod \
	allow_subdomains=true \
	max_ttl=8760h
```
Now assign a policy, you should see a file ``` cert-manager-policy.hcl ``` in your local dir:
```bash
# ls
cert-manager-policy.hcl
```
We can use it as our policy content:
```bash
vault policy write cert-manager-pki-policy cert-manager-policy.hcl
```
Apply the policy to the role:
```bash
vault write pki_int/roles/cert-manager policies=cert-manager-pki-policy
```

## Cert-manager setup
Create a namespace to run cert-manager in
```bash
kubectl create namespace cert-manager
```
Install the ___CustomResourceDefinitions___ and cert-manager itself:
```bash
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.13.0/cert-manager.yaml

```
Check to see that it's running:
```bash
kubectl get pods --namespace cert-manager

NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-6f578f4565-spm95              1/1     Running   0          20h
cert-manager-cainjector-75b6bc7b8b-984fg   1/1     Running   0          20h
cert-manager-webhook-8444c4bc77-sptf8      1/1     Running   0          20h
```
Create a secret to communicate with ___Vault___ :
```bash
kubectl create secret generic cert-manager-vault-token --from-literal=token=$VAULT_TOKEN
```
Now we create a ___Issuer___ within cert-manager: (we have to use the non-loopback address)
```bash
eth0=$(ifconfig eth0 |grep -i "inet " |awk '{print $2}'); cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1alpha2
kind: Issuer
metadata:
  name: vault-issuer
  namespace: default
spec:
  vault:
    path: pki_int/sign/cert-manager
    server: 'http://$eth0:8200'
    auth:
      tokenSecretRef:
        name: cert-manager-vault-token
        key: token
EOF
```
Validate the issuer is "online" and ready:
```bash
kubectl get issuers -o wide

NAME           READY   STATUS           AGE
vault-issuer   True    Vault verified   140m
```
Looks good, now we can actually use it. Lets say we want to create a certificate request for our wordpress site, and store the outcome in a secured object within k8s, we can set this as declarative state in a file. (locate the "wp.kernelcon2020k8s.org_cert.yaml" file in your current dir, should be "KernelCon2020K8s/Exercises/Defend/Files/defend_e12" remember)
```bash
cat wp.kernelcon2020k8s.org_cert.yaml

apiVersion: cert-manager.io/v1alpha2
kind: Certificate
metadata:
  name: wp-kernelcon2020k8s-org
  namespace: default
spec:
  secretName: wp-kernelcon2020k8s-org-tls
  issuerRef:
    name: vault-issuer
  commonName: wp.kernelcon2020k8s.org
  dnsNames:
  - wp.kernelcon2020k8s.org
```
See how we are able to define all the relevant attributes for the certificate via yaml. (feel free to edit as you see fit before deploying.)
Now deploy the "certificate request" just like any other yaml file.
```bash
# kubectl apply -f  wp.kernelcon2020k8s.org_cert.yaml
```
You can now see the certificate "state" with a command like this:
```bash
# kubectl get certs
NAME                      READY   SECRET                        AGE
wp-kernelcon2020k8s-org   True    wp-kernelcon2020k8s-org-tls   33h
```
And the cert and keys, like this:
```bash
# kubectl get secret wp-kernelcon2020k8s-org-tls -o yaml

apiVersion: v1
data:
  ca.crt: LS0tLS1CRUdJTi.....0tLS0=
  tls.crt: LS0tLS1CRUdJT......LS0=
  tls.key: LS0tLS1C.....===
kind: Secret
metadata:
  annotations:
    cert-manager.io/alt-names: wp.kernelcon2020k8s.org
    cert-manager.io/certificate-name: wp-kernelcon2020k8s-org
    cert-manager.io/common-name: wp.kernelcon2020k8s.org
    cert-manager.io/ip-sans: ""
    cert-manager.io/issuer-kind: Issuer
    cert-manager.io/issuer-name: vault-issuer
    cert-manager.io/uri-sans: ""
  name: wp-kernelcon2020k8s-org-tls
  namespace: default
type: kubernetes.io/tls
```



## Review:
Wow! that was a long one, mostly because there was a lot of setup. But no worries we will make use of vault later as well. Having a stable "secrets" backend, as well as an automated process for generating and signing certificates is important in a highly fluid environment. K8s secret storage is "ok" and (we will get into that soon) but a safer option is to store secrets in a location where they can be better encrypted and backed by a crypto device like an HSM.

## Clean up:
 None: we will make use of ___"Vault"___ in later exercises.