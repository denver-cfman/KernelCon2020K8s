# Exercise #10 - Scaling workloads
## Preface:
#### If your a developer or infrastructure type person, you may remember how difficult it was in the past to scale your applications. One had to build new servers, patch them, hope you had all the same settings and libraries installed before you could even think about installing the code. k8s makes this trivial in that it manages all the scheduling and "deploying" of your ___"code"___ or ___'workload"___ as we will see in this segment. If you can keep all your declarative descriptions of how you want your code to run "be configured" and where. Then k8s can make this very very easy.

### For visibility we will be ___"spinning up"___ a secondary console to view our pods (containers) and services. It's just a way for us to see what is going on inside of kubernetes. The kubernetes dashboard could be used for this purpose but "speak8" is just a nicer visual.

### You should still have your "wordpress" deployment running. (please check)
```bash
# kubectl get pods,svc
NAME                             READY   STATUS    RESTARTS   AGE
pod/mysql-6c8b769d74-sd5f6       1/1     Running   0          53s
pod/wordpress-6d949c5b75-6bk2h   1/1     Running   0          45s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP    23m
service/mysql-svc    ClusterIP   10.96.17.119   <none>        3306/TCP   38s
service/wp-svc       ClusterIP   10.96.93.54    <none>        80/TCP     38s
```
Now locate the "build_e10" folder, should be here: ``` KernelCon2020K8s/Exercises/Build/Files/build_e10 ``` and ```cd``` into it. There should be several yaml files ready for you to deploy.
```bash
# ls
fabric8-rbac.yml  spekt8-deployment.yml
```
Go ahead and deploy them the same way we did wordpress earlier.
```bash
# kubectl apply -f spekt8-deployment.yml
# kubectl apply -f fabric8-rbac.yml
```
Now we should have a few more pods and services in our cluster.
```bash
NAME                             READY   STATUS    RESTARTS   AGE
pod/mysql-6c8b769d74-sd5f6       1/1     Running   0          14m
pod/spekt8-754758c5fd-tzhhl      1/1     Running   0          7m54s
pod/wordpress-6d949c5b75-6bk2h   1/1     Running   0          14m

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP    37m
service/mysql-svc    ClusterIP   10.96.17.119   <none>        3306/TCP   13m
service/wp-svc       ClusterIP   10.96.93.54    <none>        80/TCP     13m
```

## Review: 
#### Foo

## Clean up: 
#### Please 
 
[Return to schedule](../../Docs/SCHEDULE.md)
