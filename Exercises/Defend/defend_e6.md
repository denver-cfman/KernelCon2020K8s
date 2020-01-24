# Managing the attack surface

## Preface: As you can imagine, the simplest way to reduce the attack surface of a running container is to remove un-needed binary files stored inside the container image. There are two ways to acomplish this.
## Post Build: 
### Say you have an image that you make use of, __node__ in this example; You need it to run your node.js app that you've built but you don't really need all that extra stuff. How would you go through and remove all the un-needed files? You could write a huge __Dockefile__ that deletes known files that you "know" you don't need, OR you could exercise the container image during a QA process and "mark" all files not touched or executed during the lifespan of your regression test. Someone else thought that was a great idea too, so they started an opensource project called [docker-slim](https://github.com/docker-slim/docker-slim) Let's build a new docker image and reduce it's attack service by using this reduction process.
- Download and set up [Dive](https://github.com/wagoodman/dive) so we can review our results.
    - Download
    ```
    # curl -k -LO https://github.com/wagoodman/dive/releases/download/v0.9.1/dive_0.9.1_linux_amd64.deb
    ```  
    - Install
    ```
    # dpkg -i dive_0.9.1_linux_amd64.deb
    ```
- Download and setup [docker-slim](https://github.com/docker-slim/docker-slim) so we can reduce the attack surface quickly.
    - Download
    ```
    # curl -k -LO https://github.com/docker-slim/docker-slim/releases/download/1.26.1/dist_linux.tar.gz
    ```
    - Install
    ```
    # tar -xzf dist_linux.tar.gz
    # cp dist_linux/* /usr/local/bin/
    rm -Rfv dist_linux
    ```
- Review the prepaired Dockerfile and adjacent files use to deploy this node.js app. (You are welcome to change this as you feel necessary as it was prepped only to make this exercise quicker)
```
# cd Exercises/Defend/Files/defend_e6_node
# ls -laSh
Dockerfile  package.json  server.js
```
- Do a docker build
```
# docker build -t localhost:5000/kernelcon-node:v0.0.1 .
```
- And test it out
```
# docker run --name node --rm -d -p 8888:8080 localhost:5000/kernelcon-node:v0.0.1
```
- Go check it out http://<kali_IP>:8888/
- spin down the image
```
# docker stop node
```
- Review the container image with dive  (Cntl+C to exit)
```
# dive localhost:5000/kernelcon-node:v0.0.1
```
### Wow! That image is huge
```
# docker images localhost:5000/kernelcon-node:v0.0.1
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
localhost:5000/kernelcon-node   v0.0.1              c387e5ac5905        2 hours ago         911MB
```
### Almost one Gig of extra "stuff". Lets see just how much is really needed to run this container image.
- Within the same dir as the existing "Dockerfile", run docker-slim
```
# docker-slim build --http-probe localhost:5000/kernelcon-node:v0.0.1
```
### This will spin-up your container use a built-in http probe to spider your "web app" and monitor all files used, creating a whitelist AND blacklist used to re-build your container image with far less content.
- look at the size diference (due to a bug, the new image name will be based on the first word in the image path)
```
# docker images |grep -i slim
localhost.slim                                           latest              eda963988494        2 minutes ago       46.7MB
docker-slim-empty-image                                  latest              4bc7c2c9d5dc        About an hour ago   0B
```
- Try out the new image
```
# docker run -d --rm -p 8888:8080 --name node localhost.slim
# docker stop node
```
- Now lets review the image itself with __dive__ (Cntl+C to exit)
```
# dive localhost.slim
```
## Wow 47mb! That is a significant reduction of attack surface!
### Granted, this will never totaly eliminate all attack surfaces but it does go a log way towards that end.

## During the build: 
### Another method to reduce the amount of attack surface is to "not add" additional file in the first place. This process is much easier for compiled content than it is for interpreted programming languages, but is still possible as you saw in the other method.

### Lets try and build a golang app "without" adding any extra un-needed files.

- Review the prepaired golang files and ajacent files use to deploy this golang app. (you are welcome to change this as you feel necessary it was preped only to make this exercise quicker)
```
# cd Exercises/Defend/Files/defend_e6_go
# ls -laSh
kernelcon.go
```
- Now create a "Dockerfile" for building this new app.
```
# touch Dockerfile
```
- Put some content in it.
```
FROM golang:latest As builder
WORKDIR /app
COPY *.go .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o kernelcon .

```
### This is our "first stage" of our build process. Yes, from Docker version 17.05 you can now have multiple "stages" in the same file. Before this version, you would have had to create multiple Dockerfiles and chain them together. i.e. FROM base:image in Dockerfile-build1 then FROM tag:from_other_Dockerfile in Dockerfile-build2 etc. In any case, we have set up our build container, copy in our golang files and compile it into a single executable.

### Now for stage two.
- Add the second stage to your Dockerfile
```

FROM scratch  
WORKDIR /root/
COPY --from=builder /app/kernelcon .
EXPOSE 8000
CMD ["./kernelcon"] 
```
- The whole file should look something like this
```
FROM golang:latest As builder
WORKDIR /app
COPY *.go .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o kernelcon .

FROM scratch  
WORKDIR /root/
COPY --from=builder /app/kernelcon .
EXPOSE 8000
CMD ["./kernelcon"] 
```
- Build it
```
# docker build --tag kernelcon-go .
```
- Now review it's size with __dive__ (Ctrl+c)
```
# dive kernelcon-go
```
### Wow! Our entire app deployed into <10mb. There is nothing else in that container! Small to store, small to update, small to upload, and small to download into a registry. 
- Give it a test drive
```
# docker run --rm -d -p 8888:8000 --name kernelcon-go kernelcon-go:latest
```
- Feel free to run our __aquasec Microscanner__ on it as well, (you won't find any vulnrabilitys)

## Review:
### We built apps using different programming languages, deployed them into container images. We validated image sizes both pre-reduction and post-reduction of container bloat. We also made use of multi-stage docker builds to compile our app "outside" the runtime container image, and, overall, reducing the attack surface drastically.

## Clean up:
```
# docker stop kernelcon-go
```

[Return to schedule](../../Docs/SCHEDULE.md)
