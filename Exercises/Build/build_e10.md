# Exercise #10 - Scaling workloads
## Preface:
#### If your a developer or infrastructure type person, you may remember how difficult it was in the past to scale your applications. One had to build new servers, patch them, hope you had all the same settings and libraries installed before you could even think about installing the code. k8s makes this trivial in that it manages all the scheduling and "deploying" of your ___"code"___ or ___'workload"___ as we will see in this segment. If you can keep all your declarative descriptions of how you want your code to run "be configured" and where. Then k8s can make this very very easy.

### For visibility we will be ___"spinning up"___ a secondary console to view our pods (containers) and services. It's just a way for us to see what is going on inside of kubernetes. The kubernetes dashboard could be used for this purpose but "speak8" is just a nicer visual.

### You should still have your "wordpress" deployment running. (please check)
```bash
# kubectl get pods,svc
NAME                             READY   STATUS    RESTARTS   AGE
pod/mysql-6c8b769d74-sd5f6       1/1     Running   0          35m
pod/wordpress-6d949c5b75-6bk2h   1/1     Running   0          35m

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP    58m
service/mysql-svc    ClusterIP   10.96.17.119   <none>        3306/TCP   35m
```
Lets say your workpress site is under more load, now that it's so popular. From a web tier, we can add more php and apache server to handle that load and serve pages faster. (We can discus strategies for Backend database servers later), for now we will focus on the front end. k8s has multiple was of handling this functionality, in fact, because

## Review: 
#### Foo

## Clean up: 
#### Please 
 
[Return to schedule](../../Docs/SCHEDULE.md)
