# Exercise #3

## Taking advantage of devlopers missconfigurations.

### Preface: "I get it!" Developers have it hard, they are expected to very rappidly and correctly churn out buesness requirements "AS CODE". Sometimes this can lead to shortcuts or just plane old "copied" sample as code just to meat a need.
![Oreilly Funny](Files/images/oreilly_funny.jpg)
Lets look at the same exercise again from the perspective of the atacker.

#### Here we go ...

- Change directory so you are in the __<REPO_ROOT>/Exercises/Break/Files/break_e3/__ directory.
- do a ```ls``` command to make sure, should see:
```
about.html      break_e3.yml    dsvw.py     index.html
```
this is the __same__ code sample as before, just copied here for compleetenes and to keep stuff separate. Anyway ~
- run the docker-compose command with our defnition file.
```
docker-compose -f break_e3.yml up -d
```
This will start up a docker container with our app running inside it, now go navagate your web browser to view it.
You can do this in many ways, but two come to mind. Use either your "host" browser or make use of the firefox browser within kali linux.

TO use your host browser, you will need to find your kali linux IP with a command comething like this:
![ifconfig eth0](Files/images/kali_ifconfig.jpg)
or just find it via the ```ifconfig eth0``` command.
Or again make use of the firefox browser within kali linux:
![kali firefox](Files/images/kali_firefox.jpg)

Then navaget to your new dev site: ```http://127.0.0.1:1234```
![kali firefox](Files/images/kali_e2_site.jpg)

Or use your Host browser if you want ```http://<kali ip>:1234/```

## Did you find it?
### Looks like there is a R.C.E. vulnrability in the "Kernelcon check" link. if you were to add a ```;``` followed by additional code; you gain code execution on the webserver. Very Bad! try it:
```
http://<your IP>:1234/?domain=kernelcon.org%3B%20ls
```
or try this one:
```
http://<your IP>:1234/?domain=kernelcon.org%3B%20echo%20%22(%E2%95%AF%C2%B0%E2%96%A1%C2%B0)%E2%95%AF%EF%B8%B5%20%E2%94%BB%E2%94%81%E2%94%BB%22%20%3E%20app%2Fabout.html
```
now go back and click on that __About us__ link.

## In and of itself, that R.C.E. IS bad, but could be contained within the container. the REAL issue steams from the fact that the developer left read/write access to there dev directory. (more on that later).

## When you are done, just pull down your dev. env. via the __docker-compose__ command again.
```
docker-compose -f break_e3.yml down
```

[Return to schedule](../../Docs/SCHEDULE.md)