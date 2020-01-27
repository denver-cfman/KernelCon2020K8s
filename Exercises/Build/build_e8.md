# Exercise #8 common tools and commands

## Preface:
### There are a wide number of tools and clients built for interaction with kubernetes. I wanted to focus on a few here because I believe they will benifit you throught this course and after when you use your own clusters. You will find that many of the day to day admin tasks as well as verifycation tasks can be reduces with many of these utilitys or tools.

| Tool Name | Purpose | Notes |
--- | --- | ---
[`kubectl`](#Kubectl) | core client | This is the Kubernetes [core client](https://kubernetes.io/docs/reference/kubectl/overview/) and is used primarly by anyone using or administration a cluster.
[`stern`](#Stern) | log review | In a cluster you may need to review logs "across" multiple running instances of containers, this [__third party__](https://github.com/wercker/stern) tool make it a little easier. 
[`helm`](#Helm) | deployment client | As you find you can deploy objects into k8s via yaml files, althow conveanent, [Helm](https://helm.sh/docs/intro/quickstart/) allows you to keep track of, rollback and package multiple containers or "deployments" into your cluster. You could think of it as __apt__ or __yum__ for your cluster but you package up your deployment.
[`kui`](#Kui) | kubectl plugin | This [third party extention](https://github.com/IBM/kui) is an interactive shell between you and kubectl, In addition you can use it to visulize your cluster, pods, deployment anything really.
[`popeye`](#Popeye) | cluster security | This third party tool is designed to help you review the security posture of your cluster.


## Kubectl
### Foo Bar

## Stern
### FooBar Baz

## Helm
### Foo

## Kui
### Foo

## Popeye
### Baz

