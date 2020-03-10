# Vault Setup

## Container setup (via docker-compose)

### 

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
Your values will be random and unique, copy them and save them for later.
```bash
# curl -k -vv http://127.0.0.1:8200/v1/sys/seal-status
```

export VAULT_TOKEN=$(docker-compose -f docker-compose.yaml logs |grep -e "Root Token" |cut -d ":" -f 2 |tr -d " ")

vault secrets enable pki
vault secrets tune -max-lease-ttl=8760h pki

kubectl create secret generic cert-manager-vault-token --from-literal=token=$VAULT_TOKEN

kubectl create namespace cert-manager

$ kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/v0.13.1/deploy/manifests/00-crds.yaml

kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.13.1/cert-manager.yaml