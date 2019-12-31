# Our First Workload

## Once you have validated that docker is installed and currently running
```bash
$ docker info
```
## we will download a sample container and verify that it is running within your lab environment.

First we will download the container image:
```bash
$ docker pull kennethreitz/httpbin
```
if sucsesful, you should now see this image in your "container runtime" image inventory.
```bash
$ docker images
```
the output will look something simior to this:
```bash
python                                 <none>              5b0283c5034b        5 months ago        169MB
python                                 <none>              4ae385ba9dd2        5 months ago        909MB
nginx                                  <none>              e445ab08b2be        5 months ago        126MB
kennethreitz/httpbin                   latest              b138b9264903        14 months ago       534MB
```
Now we can "run" this image by involking our "container runtime" like this, with some basic options.
```bash
$ docker run -d --rm --name httpbin -p 8888:80 kennethreitz/httpbin
```
This will start the "httpbin" image, map a local tcp port "8888" to it's inner tcp port "80" and give it a name of "httpbin".

Command breakdown

Option | Meaning | Note
--- | --- | ---
run | start or "run" the container | if you do not "pull" the image first, "run" will also pull the image as well.
-d | run in deamon mode | non interactive, run in background.
--rm | remove the runtime container after it is stopped | if ommited you would have to do "*docker stop httpbin*" AND "*docker rm httpbin*" to free resources etc.
--name | name this container | if ommited the runtim will make up an odd name that you will have to search for via "*docker ps -a*" before you can "*docker stop*" or "*docker rm*"
-p | port mapping | take the local tcp port *8888* and map it to the container network port of *80*
kennethreitz/httpbin | the name of the image to *run* | you may also defign a version like *kennethreitz/httpbin:latest* or speciffic hash *kennethreitz/httpbin:sha256:b138b9264903f46a43e1c750e07dc06f5d2a1bd5d51f37fb185bc608f61090dd* this can be helpful if you need to *pin* a very speciffic version of an image to be used (recomended)

### Now that your test workload is up and running, try to access it.
```bash
curl -k -v http://127.0.0.1:8888/get
```
your output should be simalor to this:
```bash
< 
{
  "args": {}, 
  "headers": {
    "Accept": "*/*", 
    "Host": "127.0.0.1:8888", 
    "User-Agent": "curl/7.64.1"
  }, 
  "origin": "172.17.0.1", 
  "url": "http://127.0.0.1:8888/get"
}
* Connection #0 to host 127.0.0.1 left intact
* Closing connection 0
```
Try using your web browser, explore, play around with your new service!

### Once you finished with your workload, please shut it down, and remove it.
```bash
$ docker stop httpbin
```
this will free up docker resources, so you can move on to other exercises.

## Congradulations, you have compleeted your first exercise "Our First Workload"
