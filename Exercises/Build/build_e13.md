# Exercise #13 (Ingress)

## Preface:
![kube-proxy](/Docs/Images/k8s_networking_port-forward.png)
Up until now, we have been making use of the k8s ___kube-proxy___ functionality to see our services and pods. In the real world clusters typically expose network ports via something called a ___nodePort___
lets look at what a node and pod look like without any ___nodePorts___ setup.
```bash
# kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
httpbin-85d57ddd75-qp69k   1/1     Running   0          17h   172.17.0.6   minikube   <none>           <none>
httpbin-85d57ddd75-tqfdz   1/1     Running   0          17h   172.17.0.5   minikube   <none>           <none>
```
![node-net](/Docs/Images/k8s_networking_node_view.png)



## Review:

## Clean up:

[Return to schedule](../../Docs/SCHEDULE.md)