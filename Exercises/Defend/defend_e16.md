# Exercise 16 (Integrated Defense)

## Preface:
In my opinion, defense strategies should be coupled with the thing that they are meant to defend. In this example we have a simple web application, one that we know is risky or may be vulnerable to deploying. Therefore we can implement a defense control out in fount of this server. Something that doesn't interfere with the App or how it works, even how it's deployed. 

The design of this exercise is to show you that you can integrate perimeter security into kubernetes and your deployments.

You should still have your wordpress app and database still running in your cluster.
```bash
kubectl get pods

NAME                         READY   STATUS    RESTARTS   AGE
httpbin-7b5478bc48-g7sxj     1/1     Running   0          81m
httpbin-7b5478bc48-n78cs     1/1     Running   0          81m
mysql-6c8b769d74-9v5n2       1/1     Running   0          2m27s
wordpress-6d949c5b75-wj6cf   1/1     Running   0          2m27s
```

- Lets kill the "httpbin" stuff (if you haven't done so already)
```bash
kubectl delete ing rewrite
kubectl delete svc httpbin-svc
kubectl delete deployment httpbin
```
Note: notice how we remove resources and objects in the opposite order we created them!

For now we want to keep all the other ingress-controller resources, FYI as soon as we removed the "Ingress" object all the ___kube-proxy___ routing went away. If you looked at [https://k8s.kernelcon2020.org/](https://k8s.kernelcon2020.org) you would get a 404, because there is no routing in place to receive the traffic. 


```bash

cat <<EOF | kubectl apply --filename -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: wordpress
  namespace: default
spec:
  tls:
    - hosts:
      - k8s.kernelcon2020.org
      secretName: star-kernelcon2020-org-tls
  rules:
  - host: k8s.kernelcon2020.org
    http:
      paths:
      - backend:
          serviceName: wp-svc
          servicePort: 80
        path: /
EOF

```
You should be able to access the wordpress site via the FQDN we setup in the last exercise [http://k8s.kernelcon2020.org](http://k8s.kernelcon2020.org)


```bash

cat <<EOF | kubectl apply --filename -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/enable-modsecurity: "true"
    nginx.ingress.kubernetes.io/modsecurity-snippet: |
      SecRuleEngine On
      SecRequestBodyAccess On
      SecAuditEngine RelevantOnly
      SecAuditLogParts ABIJDEFHZ
      SecAuditLog /var/log/modsec_audit.log
      SecRule REQUEST_HEADERS:User-Agent "kernelcon-scanner" "log,deny,id:107,status:403,msg:\'KernelCon Scanner Identified\'"
  name: wordpress
  namespace: default
spec:
  tls:
    - hosts:
      - k8s.kernelcon2020.org
      secretName: star-kernelcon2020-org-tls
  rules:
  - host: k8s.kernelcon2020.org
    http:
      paths:
      - backend:
          serviceName: wp-svc
          servicePort: 80
        path: /
EOF

```

- now lets use ```curl``` to hit the site, this should trigger the WAF and record an event.

```bash
curl -k -vv -H "User-Agent: kernelcon-scanner" https://k8s.kernelcon2020.org/
```

- now check the logs, you should see your events.

```bash
### Log file review
kubectl exec -it -n ingress-nginx $(kubectl -n ingress-nginx get pods -o=jsonpath='{.items[0].metadata.name}') cat /var/log/modsec_audit.log

```

#### Note: mod_security ,by default is setup to monitor triggered events. Core Rule Set can easily be enabled by adding the ___nginx.ingress.kubernetes.io/enable-owasp-modsecurity-crs: "true"___ annotation. If you want to expand on this technique, have a look at the [module](https://modsecurity.org/crs/) to discover way you can integrate blocking and other custom rules.


## Review:
There are many open source projects contributing and integrating code into the microservices ecosystem. Likely you fine one specific to the needs of your code and may already have a kubernetes setup or configuration. I've shied away form calling these "bolt-on" solutions and instead prefer to call them modular security improvements.


## Clean up:
FOr now let's just kill the "Ingress" object mapped to our wordpress service.
```bash
kubectl delete ing wordpress
```

 [Return to schedule](../../Docs/SCHEDULE.md)