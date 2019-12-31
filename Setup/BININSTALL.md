# PlaceHolder BININSTALL.md

Notes: Kali (Docker Install Guide [Here](https://medium.com/@airman604/installing-docker-in-kali-linux-2017-1-fbaa4d1447fe)
       Kubectl (Install Note) 

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```

## Docker Compose
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

```


