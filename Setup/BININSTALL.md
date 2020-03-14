# If you are using your own copy of Kali Linux, you may wish to install the needed files manually.

## To do so please use these instructions.

### The needed binaries are:
- kubectl
- docker
- docker-compose
- minikube
- popeye
- krew
- k3s

<br />
<br />
<br />

## [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)
### Note: This is the "Fat Client" used universally to easily interact with the kubernetes api server.
#### To install simply download the binary and place it in your path.
```
# sudo curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```
Then
```
# sudo chmod +x kubectl && cp kubectl /usr/local/bin/
```
<br />
<br />
<br />

## [docker](https://www.docker.com/)
### Note: This is the most popular container _runtime_, and we will need it to run individual container images.
#### To install simply run these commands to add the docker repo to your apt sources, ensure you don't have an existing copy of docker installed; then install the docker-ce version via the apt-get command.
```
# sudo curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -

# sudo echo 'deb [arch=amd64] https://download.docker.com/linux/debian buster stable' > /etc/apt/sources.list.d/docker.list

# sudo apt-get update

# sudo apt-get remove docker docker-engine docker.io

# sudo apt-get install docker-ce

# sudo systemctl start docker

# sudo systemctl enable docker
```
Then test it
```
# sudo docker run --rm hello-world
```
<br />
<br />
<br />

## [Docker Compose](https://docs.docker.com/compose/)
### Note: docker-compose is the primary _Fat Client_ used to _spin up_ one or more containers. In addition it's used with docker _swarm_.
#### To install simply download the binary, add to your path.
```
# sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# sudo chmod +x /usr/local/bin/docker-compose

# sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

```

<br />
<br />
<br />

## [Minikube](https://minikube.sigs.k8s.io/)
### Note: minikube is just a fancy script that can _setup_ a single node kubernetes cluster, interact with it and complete some of the more complicated tasks easily.
#### To install simply download the binary, add to your path.
```
# sudo curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# sudo chmod +x minikube

# sudo cp minikube /usr/local/bin/
```

<br />
<br />
<br />

## [Popeye](https://github.com/derailed/popeye)
### Note: Popeye is a utility that scans live Kubernetes cluster and reports potential issues with deployed resources and configurations.
#### To install simply download the binary, add to your path.
```
# sudo curl -LO https://github.com/derailed/popeye/releases/download/v0.6.1/popeye_0.6.1_Linux_x86_64.tar.gz

# sudo tar zxvf popeye_0.6.1_Linux_x86_64.tar.gz

# sudo chmod +x popeye

# sudo cp popeye /usr/local/bin/
```

<br />
<br />
<br />

## [Krew](https://github.com/kubernetes-sigs/krew/)
### Note: As powerful as _kubectl_ is, sometimes it needs some help, therefore __plug-ins__ have been created. Krew is a *package manager* of kubectl plug-ins.
#### To Install, run these two commands. __(as root user)__
```
(
  set -x; cd "$(mktemp -d)" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/download/v0.3.3/krew.{tar.gz,yaml}" &&
  tar zxvf krew.tar.gz &&
  KREW=./krew-"$(uname | tr '[:upper:]' '[:lower:]')_amd64" &&
  "$KREW" install --manifest=krew.yaml --archive=krew.tar.gz &&
  "$KREW" update
)
```
Then add the .krew folder to your path
```
# export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"

# echo 'export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"' >> ~/.bashrc
```

<br />
<br />
<br />

## [k3s](https://k3s.io)
### Note: Created by __[Rancher](https://rancher.com/)__ this little binary contains kubernetes, a container runtime AND all the fixings. Used to setup a kubernetes cluster with a small foot print it's well suited for low memory systems like edge IOT devices (like the [Raspberry Pi](https://www.raspberrypi.org/products/))
#### To install, simply run the installer script.  __(as root user)__
```
# curl -sfL https://get.k3s.io | sh -
```
