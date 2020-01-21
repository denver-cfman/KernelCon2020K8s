# Exercise #5

# Image packaging, scanning, and storage

## Preface: 

### Container runtime engines will locally store what are called "images" once you have "fetched" them (remembering the ___docker pull___ exercise from before, when an "image" file was pulled down) and these, then, are essentially "packages" of manifests and compressed files that all make up what is "mounted" by the container runtime. 
### There are many commands which deal with images directly (i.e. ___docker pull___, ___docker diff___, ___docker inspect___, ___docker save___, ___docker load___, to name a few). Right now, we will be looking at what it takes to create an image, store the image, and retrieve it. 

### Docker makes this easy with a scripted interface called "[Dockerfile](https://docs.docker.com/engine/reference/builder/)". Basically, the docker client will "read in" each line of this script, and interpret it into specific actions needed to create or refine a specific image.

## Let's make an image

- Make a new folder and create a new "Dockerfile"
```
# mkdir E5 && cd E5 && touch Dockerfile
```
- Edit the "Dockerfile" so it's contents look like this:
```
FROM ubuntu:16.04
CMD ["/bin/echo","Hello, KernelCon 2020!"]
```
#### Note: Yes, I know that using "ubuntu:16.04" is WAY "overkill" for this image, but it is a setup for later :-)
- Then, we build it
```
# docker build --tag hello-kernelcon .
```
- Then we test run it:
```
# docker run --rm hello-kernelcon
```
### What the heck just happened?
### Well, we instructed docker to "pull down" the "__latest__" image of __ubuntu__ (as a tar ball) and use it as the *base* of our new image, then we just used an existing binary which was already present in the image "__echo__" (in this case), to display a message.

### As we __ADD__ or __COPY__ content into our containers, they are stored as additional *layers* in the *overlay* filesystem that makes up each __image__, and they become available to any process running within that container.   Let's do a very simple example:
- Edit your __Dockerfile__ again to look like this:
```
FROM ubuntu:16.04
WORKDIR /
COPY foo.txt .
CMD ["/bin/echo","Hello, KernelCon 2020!"]
```
- Now, you need to make a new file
```
 # echo "Hi there, KernelCon 2020!" >> foo.txt
```
- You need to __build__ it again via docker:
```
docker build --tag hello-kernelcon .
```
### This time, let's take a look at the __history__ of the image:
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
### You should see the file copy within one of the new __layers__ you just created. Now, let's interact with it by __overriding__ the default command __/bin/echo__ (This is what you set in the earlier build).
```
# docker run --rm hello-kernelcon /bin/cat /foo.txt
```
### We will read the file __foo.txt__ from inside the container image using a process __cat__ from inside the container image.

# Storage

### Now, that's all well and good, but these __images__ are not worth much if we can't share them between systems or with each other. Enter "the registry". Most people start with "official" images stored within the DTR (Docker Trusted Registry), or Docker Hub. However, there is nothing stopping you from using ___docker save___ on one system, then ___docker load___, on another. In fact, this is exactly how many "air-gapped" systems need to work. But, in most cases, developers set up their own registry to store their images. A few of note are:
- [Docker Hub]()
- [GitLab Container Registry]()
- [Docker Registry](https://hub.docker.com/_/registry)
- [JFrogs Container Registry](https://jfrog.com/container-registry/)
- [Sonatype Nexus Repository Manager (OSS)](https://www.sonatype.com/nexus-repository-oss)
- and many, many, more...

## Let's set up our own registry
- Run the docker command to "pull down", "start", and "host" a local registry.
```
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```
## Remote storage
### Let's take that __hello-kernelcon__ image from earlier in the exercise, and store it in our remote registry.
- Tag the image for remote storage
```
# docker tag hello-kernelcon:latest localhost:5000/hello-kernelcon:latest
```
- Then __Push__ the image up into the remote registry
```
# docker push localhost:5000/hello-kernelcon:latest
```
- Now, just for "grins and giggles", let's remove all of the local versions of this image and __pull it back down again__
```
# docker rmi localhost:5000/hello-kernelcon:latest

# docker rmi hello-kernelcon:latest
```
- Now, you can download it and run it again (this time from the remote registry)
```
# docker run --rm --name kernelcon localhost:5000/hello-kernelcon:latest
```

# Scanning

### As you have seen, "images" are the base for all content used during execution of a container, and are essential to container use. As you may already guess, trusting the content of this image is essential to creating a secure environment. Therefore, we must constantly update, and periodically  __scan__, the content of these __images__ to ensure they are both safe and usable.  Some registries will automatically do this. ___Nexus___, ___JFrog___, and ___Docker Hub___ do this with automated bots that scan images already stored in their registries. CI/CD systems can be made to review the results of these scanning tools' outputs and stop builds on "bad" scores. Or, you can review your output manualy, or "script up" a tool of your own.

### Manual scan walkthrough. Let's take our example image we just created; All we did was add a test file to the image...it should be safe, right? We didn't ADD any scary new "Zero-Day" malware into the image, so, we should be able to use it, right? Let's see.

- For this exercise you may want to have __jq__ installed.
```
# apt install jq
```

- Navigate into your build_5e folder.
```
# cd Exercises/Build/Files/build_e5/
```
- GitHub user "[lukebond](https://github.com/lukebond)" has created a great "[wrapper](https://github.com/lukebond/microscanner-wrapper)" script that will take an exsisting tool "[Microscanner](https://github.com/aquasecurity/microscanner) By [Aquasec](https://www.aquasec.com/)" and make it non intrusive to the container-building process.
- First we pull in the latest version on this "wrapper script."
```
# curl -k -LO https://github.com/lukebond/microscanner-wrapper/archive/master.zip
```
- Then unzip
```
# unzip master.zip
```
- Then change into the repo dir
```
cd microscanner-wrapper-master/
```
- Now, let's test that image file we just created, and use the access token you received from _Aquasec_ and put the output into a json file...this will take a moment to run.
```
echo "{" >> hello-kernelcon.json && USE_LOCAL=1 MICROSCANNER_TOKEN=xxxxxxxxxxxxxxxx ./grabjson.sh localhost:5000/hello-kernelcon:latest >> hello-kernelcon.json 
```
#### Note: output can be put into html files or integrated with CI/CD systems as well to automate this type of scanning.
- Now, view the results
```
cat hello-kernelcon.json |jq '.resources[].vulnerabilities[] | [.name,.description]'
```

### Wait!...What?...All we did was add a text file!... Why are there so many vulnerabilities?
#### When we built our image, we made use of a large "distro" base __ubuntu__. You will see in our next exercise, techniques used to reduce, or in some cases completely remove, additional un-needed files that clutter up our deployable images.

# Review:
### In this exercise we went briefly into 1) how to create container images, 2) the storage of container images (both local and remote), as well as 3) scanning them once they are built. These are core concepts to know as we transition from "container-specific" exercises into "orchestration-specific" exercises.

## Clean up:
#### Please leave your local registry running, as well as the scanning files "in-place", as we will make use of them in future exercises.
 
[Return to schedule](../../Docs/SCHEDULE.md)
