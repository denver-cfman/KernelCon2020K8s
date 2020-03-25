# Exercise 21 (Attacking secret storage)

## Preface:
If your targets are ___"mounting"___ secrets inside of containers, you can generally find them in two places.

- In the file system
- In enlivenment variables

### You should still have the 

```bash

kubectl exec -it $(kubectl get pods -l app=mysql -o=jsonpath='{.items[0].metadata.name}') /bin/bash

```

- Once in the container you can run something like this.

```bash
root@mysql-6c8b769d74-g269z:/# env |grep -i pass

MYSQL_PASSWORD=###############
MYSQL_ROOT_PASSWORD=xxxxxxxx

### or in the File system like this.

root@mysql-6c8b769d74-g269z:/# ls -R /run/secrets/
/run/secrets/:
kubernetes.io

/run/secrets/kubernetes.io:
serviceaccount

/run/secrets/kubernetes.io/serviceaccount:
ca.crt  namespace  token

root@mysql-6c8b769d74-g269z:/# cat /run/secrets/kubernetes.io/serviceaccount/ca.crt    
-----BEGIN CERTIFICATE-----
MIIC5zCCAc+gAwIBAgIBATANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwptaW5p
a3ViZUNBMB4XDTIwMDMyNDA1MDYyNloXDTMwMDMyMzA1MDYyNlowFTETMBEGA1UE
AxMKbWluaWt1YmVDQTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKp4
```

### Yes thats right lots of interesting stuff is ___"mounted"___ inside the container at run time.

## Review:
Depending on the constraints admins have put on pods and hardening of the containers in general, you can gleam a lot of information from inside the container and use that info to laterally move within the cluster.

## Clean up:

None

 [Return to schedule](../../Docs/SCHEDULE.md)