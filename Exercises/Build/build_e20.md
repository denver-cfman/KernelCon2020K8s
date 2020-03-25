
# Exercise 20 (How do secrets work)

## Preface:
     Early on in k8s development is was obvious that some kind of secret management would be required to "copy" or inject secret strings or data into the runtime of containers or other resources.

     When starting the api-server options can be used to extend or enforce how k8s does this. In a generic configuration secrets are encoded and then stored inside the runtime database used foe K8s (etcd). The argument called ``` --encryption-provider-config ``` can be passed to api-server on setup to change how it's encrypted. All keys stored in the "EncryptionConfiguration" kind are OK'ish but the only real production ready solution is to attach kubernetes to a real crypto backend via a [KMS Providor](https://en.wikipedia.org/wiki/Key_management).

     Here we will review how to use secrets in k8s.

- This is what a secret manifest looks like.

```bash
apiVersion: v1
kind: Secret
metadata:
name: kernelcon-secret
data:
username: YWRtaW4=
password: cGVhay1hLWJvbw==
```

If you don't recognize the format, thats [base64 encoded](https://codebeautify.org/base64-decode) ascii strings.

If you recall we added a secret via kubectl earlier when we setup wordpress and mysql.
```bash
kubectl create secret generic mysql-root-pass --from-literal=password='password'

kubectl create secret generic mysql-pass --from-literal=password='password'

```

- and looks like this

```bash
# kubectl get secret mysql-root-pass -o yaml

apiVersion: v1
data:
  password: cGFzc3dvcmQ=
kind: Secret
metadata:
  creationTimestamp: "2020-03-25T05:08:51Z"
  name: mysql-root-pass
  namespace: default
  resourceVersion: "620"
  selfLink: /api/v1/namespaces/default/secrets/mysql-root-pass
  uid: 882499b2-0c4d-4ede-8630-3a95ab32a161
type: Opaque
```
#### Right now this secret is only stored in etcd, and protected by encryption (again keys stored in k8s as well) and accessible only to those with the ___"list"___ and ___"watch"___ requests against the ___"secrets"___ resource in the API.

- you SHOULD consider anyone with list and watch on secrets to have FULL access to these secrets.

- because it's this easy to reverse them!!!

```bash

echo "cGFzc3dvcmQ=" | base64 -d

```


## Review:

In general we took a quick look at secrets in kubernetes, later we will expand on this with additional exercises. 

## Clean up:
None, we will use it in a later exersize.

 [Return to schedule](../../Docs/SCHEDULE.md)