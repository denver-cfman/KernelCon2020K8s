# Exercise #5

# Image packaging, scanning and storage

## Preface: 

### Container runtime engins will store what they call "images" locally once you have "fetched" them, remember our ___docker pull___ exercise from before, we pulled down an image file a "package" of manafests and compressed files that all make up what is "mounted" by the container runtime. 
### There are many commands that deal with images directly ___docker pull___, ___docker diff___, ___docker inspect___, __docker save__, __docker load__ to name a few. Right now we will look at what it takes to create an image, store and retreave them. 

### Docker makes this easy with scripted inface called "[Dockerfile](https://docs.docker.com/engine/reference/builder/)". Basicly the docker client will read in each line of this script and intrurpret it into spsciffic actions needed to create or refign a speciffic image.

## Let's make one

- make a new folder and create a new "Dockerfile"
```
# mkdir E5 && cd E5 && touch Dockerfile
```
- Edit the "Dockerfile" so it's contents look like this:
```
FROM ubuntu:16.04
CMD ["/bin/echo","Hello KernelCon 2020!"]
```
#### Note: I know using "ubuntu:16.04" is WAY over kill for this image, it's a setup for later :-)
- Then we build it
```
# docker build --tag hello-kernelcon .
```
- Then we test run it:
```
# docker run --rm hello-kernelcon
```
## What the hell just happined?
## Well we instructed docker to pull down the "__latest__" image of __ubuntu__ (a tar ball) and use it as the *base* of our new image, then we just used an exsisting binary already present in the image __echo__ (in this case) to display a message.

## As we __ADD__ or __COPY__ content into our containers they are stored as aditional *layers* in the *overlay* filesystem that makes up each __image__ and they become available to any process running within that container, lets do a very simple example:
- edit your __Dockerfile__ again to look like this:
```
FROM ubuntu:16.04
WORKDIR /
COPY foo.txt .
CMD ["/bin/echo","Hello KernelCon 2020!"]
```
- now we need to make a new file
```
 # echo "Hi There KernelCon 2020" >> foo.txt
```
- we need to __build__ it again via docker:
```
docker build --tag hello-kernelcon .
```
## This time lets take a look at the __history__ of the image:
```
# docker history hello-kernelcon

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
71aea069adef        2 seconds ago       /bin/sh -c #(nop)  CMD ["/bin/echo" "Hello K…   0B                  
df31d827354d        2 seconds ago       /bin/sh -c #(nop) COPY file:e211b2a00deb5ef9…   24B                 
05c1c99a464a        3 seconds ago       /bin/sh -c #(nop) WORKDIR /                     0B                  
96da9143fb18        4 days ago          /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>           4 days ago          /bin/sh -c mkdir -p /run/systemd && echo 'do…   7B                  
<missing>           4 days ago          /bin/sh -c set -xe   && echo '#!/bin/sh' > /…   745B                
<missing>           4 days ago          /bin/sh -c rm -rf /var/lib/apt/lists/*          0B                  
<missing>           4 days ago          /bin/sh -c #(nop) ADD file:4b2eb5cd0b37ca015…   124MB  
```
## you should see the file copy within one of the new __layers__ we just created. Now lets interact with it by __overriding__ the default command __/bin/echo__ (what we set in the earlier bild).
```
# docker run --rm hello-kernelcon /bin/cat /foo.txt
```
## we read the file __foo.txt__ from inside the container image using a process __cat__ from inside the container image.

# Storage

## Now thats all well and good, but these __images__ are not worth much if we can share them between systems or with each other. Enter the "registry". Most people start with "official" images stored within the DTR (Docker trusted Registry) or (Docker Hub). However there is nothing form stopping you from using __docker save__ on one system then __docker load__ on another, in fact this is how many "air gaped" systems need to work. But in most caces people setup there own registry to store there images on. A few of note are:
- [Docker Hub]()
- [GitLab Container Registry]()
- [Docker Registry](https://hub.docker.com/_/registry)
- [JFrogs Container Registry](https://jfrog.com/container-registry/)
- [Sonatype Nexus Repository Manager (OSS)](https://www.sonatype.com/nexus-repository-oss)
- and many many more  ...

## Let us setup our own
- run the docker command to "pull down", "start" and host a local registry.
```
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```
## Remote storage
### Lets take that __hello-kernelcon__ image from earliewr and store it in our remote registry.
- Tag the image for remote storage
```
# docker tag hello-kernelcon:latest localhost:5000/hello-kernelcon:latest
```
- Then __Push__ the image up into the remote registry
```
# docker push localhost:5000/hello-kernelcon:latest
```
- now for "grins and giggles" lets remove all local versions of this image and __pull it back down again__
```
# docker rmi localhost:5000/hello-kernelcon:latest

# docker rmi hello-kernelcon:latest
```
- Now we can download and run it again (this time from the remote registry)
```
# docker run --rm --name kernelcon localhost:5000/hello-kernelcon:latest
```

# Scanning

## As you have seen, Images are the base for all content used durring execution of a container and are esentul to container use. As you can already guess trusting the content of this image is paramount to a secure environment. Therefore we must constently update and peroticy __scan__ the content of these __images__ to ensure they are safe and usable.  Some registrys will automaticly do this. Nexus, JFrog and Docker Hub do this with automated bots that scan images already stored in there registrys. CI/CD systems can be made to review the results of these scanning tools output and stop builds on bad scores. Or you can review output manualy or script up a tool of your own.

### Manual scan walkthrough. Lets take our exsample image we just created; all we did was add a test file to the image, it should be safe right, we dident ADD any scarry new 0-Day malware into the image, we should be able to use it right? Lets see.

- For this exercise you may want to have __jq__ installed.
```
# apt install jq
```

- Navagate into your build_5e folder.
```
# cd Exercises/Build/Files/build_e5/
```
- GitHub user "[lukebond](https://github.com/lukebond)" has created a great "[wrapper](https://github.com/lukebond/microscanner-wrapper)" script that will take an exsisting tool "[Microscanner](https://github.com/aquasecurity/microscanner) By [Aquasec](https://www.aquasec.com/)" and make it non intrusave to the container building process.
- first we pull in the latest version on this "wrapper script."
```
# curl -k -LO https://github.com/lukebond/microscanner-wrapper/archive/master.zip
```
- then unzip
```
# unzip master.zip
```
- then change into the repo dir
```
cd microscanner-wrapper-master/
```
- now lets test that image file we just created, use the access token you recieved from aquasec and put the output into a json file. (this will take a moment to run)
```
echo "{" >> hello-kernelcon.json && USE_LOCAL=1 MICROSCANNER_TOKEN=xxxxxxxxxxxxxxxx ./grabjson.sh localhost:5000/hello-kernelcon:latest >> hello-kernelcon.json 
```
### Note: output can be put into html files or integrated with CI/CD systems as well to automate this type of scanning.
- now view the results
```
cat hello-kernelcon.json |jq '.resources[].vulnerabilities[] | [.name,.description]'
```

## Wait, What, all we did was add a text file, why are there so many vulnrabilitys?
