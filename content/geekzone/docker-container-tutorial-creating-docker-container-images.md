---
title: Docker Container Tutorial - Creating Docker Container Images
description: ""
author: Ganesh Velrajan
tags: [
    GeekZone, Docker, Kubernetes
]
date: 2018-07-19
categories: [
    GeekZone, Docker
]
images: ["/images/docker/docker.png"]
---

## How to create Docker images:

In this tutorial we'll see how to create Docker container images.  There are two methods to create a docker image:

-   Manual method
-   Dockerfile method

### Manual Method:

First let's pull and run a latest version of ubuntu container.

    $ docker container run -it ubuntu
    Unable to find image 'ubuntu:latest' locally
    latest: Pulling from library/ubuntu
    6b98dfc16071: Pull complete 
    4001a1209541: Pull complete 
    6319fc68c576: Pull complete 
    b24603670dc3: Pull complete 
    97f170c87c6f: Pull complete 
    Digest: sha256:5f4bdc3467537cbbe563e80db2c3ec95d548a9145d64453b06939c4592d67b6d
    Status: Downloaded newer image for ubuntu:latest
    root@b549dd9ccb9e:/# 

Let's modify the contents of the container root file system by creating a new file named "testfile".

    root@b549dd9ccb9e:/# echo "It's fun to learn Docker containers" > testfile 
    root@b549dd9ccb9e:/# cat testfile 
    It's fun to learn Docker containers 
    root@b549dd9ccb9e:/# exit 
    exit 
    $ docker ps -a | grep ubuntu 
    b549dd9ccb9e ubuntu "/bin/bash" 3 minutes ago Exited (0) 3 minutes ago agitated_kepler $

Now let's check what changes we have made in the ubuntu container using the following diff command.

    $ docker container diff b549
    C /root
    A /root/.bash_history
    A /testfile
    $

We could commit the changes we have made in this ubuntu container and make it an image.

    $ docker container commit b549
    sha256:ce1fc66062791dbda9056d74ad5873a53dffd558ac4aa23bd44e51af8fd8b850
    $ docker image ls 
    REPOSITORY TAG IMAGE ID CREATED SIZE
      ce1fc6606279 23 seconds ago 81.2MB
    ubuntu latest 113a43faa138 2 weeks ago 81.2MB

The commit assigned a unique image id(ce1fc6606279) to the modified ubuntu image.  Let's associate a name "*funtolearn*" with the image.

    $ docker image tag ce1f funtolearn:latest
    $ docker image ls
    REPOSITORY TAG IMAGE ID CREATED SIZE
    funtolearn latest ce1fc6606279 10 minutes ago 81.2MB
    ubuntu latest 113a43faa138 2 weeks ago 81.2MB

We could now distribute this image to others so that they could spawn a container using this image and it would contain the *testfile* we created.

    $ docker container run -it funtolearn
    root@665193b64648:/# ls
    bin dev home lib64 mnt proc run srv testfile usr
    boot etc lib media opt root sbin sys tmp var
    root@665193b64648:/# cat testfile 
    It's fun to learn Docker containers
    root@665193b64648:/# exit
    exit
    $

To summarize, what we did in this exercise was we pulled a base version of ubuntu container and then manually added a file.  We then created a new docker image by committing this modified container workspace and gave it a name.

### Dockerfile Method:

First let's create our *testfile* and add some contents to it.

    $ echo "It's SO MUCH FUN to learn Docker containers" > testfile

Next let's create a *Dockerfile* containing the following instructions to build a Docker image.

    $ cat Dockerfile 
    FROM ubuntu
    COPY ./testfile /var/local
    WORKDIR /var/local
    CMD ["cat", "testfile"]

Here is what the above Dockerfile asks Docker to do:  Take the latest version of ubuntu image as the base, copy the *testfile* in the host directory to the container's /var/local directory, change the working directory of the container to /var/local and finally execute the command "cat testfile".

Let's create the docker image using the below command.

    $ docker image build -t muchfuntolearn .
    Sending build context to Docker daemon 17.92kB
    Step 1/4 : FROM ubuntu
     ---> 113a43faa138
    Step 2/4 : COPY ./testfile /var/local
     ---> ed771f3dc583
    Step 3/4 : WORKDIR /var/local
    Removing intermediate container b7ba093f07d0
     ---> df81918abdca
    Step 4/4 : CMD ["cat", "testfile"]
     ---> Running in 7e782fecd2cb
    Removing intermediate container 7e782fecd2cb
     ---> b192a036aa94
    Successfully built b192a036aa94
    Successfully tagged muchfuntolearn:latest

The "." in the above command specifies the PATH to find the "Dockerfile".  Let's check if the image has been created in the name.

    $ docker image ls
    REPOSITORY TAG IMAGE ID CREATED SIZE
    muchfuntolearn latest b192a036aa94 15 seconds ago 81.2MB
    funtolearn latest ce1fc6606279 About an hour ago 81.2MB
    ubuntu latest 113a43faa138 2 weeks ago 81.2MB

If we run the container using the newly created image we'll see the contents of the *testfile* getting displayed.

    $ docker container run muchfuntolearn
    It's SO MUCH FUN to learn Docker containers

The beauty of the Dockerfile method is that it enables us to build Docker images with many variations quickly by just editing the Dockerfile.

An interesting fact about Docker images is that it is made of many layers.  We could display the various layers in an image, for eg: the base ubuntu image, using the following command.

    $ docker image history ubuntu
    IMAGE CREATED CREATED BY SIZE COMMENT
    113a43faa138 2 weeks ago /bin/sh -c #(nop) CMD ["/bin/bash"] 0B 
     2 weeks ago /bin/sh -c mkdir -p /run/systemd && echo 'do… 7B 
     2 weeks ago /bin/sh -c sed -i 's/^#\s*\(deb.*universe\)$… 2.76kB 
     2 weeks ago /bin/sh -c rm -rf /var/lib/apt/lists/* 0B 
     2 weeks ago /bin/sh -c set -xe && echo '#!/bin/sh' > /… 745B 
     2 weeks ago /bin/sh -c #(nop) ADD file:28c0771e44ff530db… 81.1MB

Let's check if the image we created has more layers added on top of the ubuntu image layers.

    $ docker image history muchfuntolearn
    IMAGE CREATED CREATED BY SIZE COMMENT
    b192a036aa94 32 minutes ago /bin/sh -c #(nop) CMD ["cat" "testfile"] 0B 
    df81918abdca 32 minutes ago /bin/sh -c #(nop) WORKDIR /var/local 0B 
    ed771f3dc583 32 minutes ago /bin/sh -c #(nop) COPY file:41c23f7a0a3b38c1… 44B 
    113a43faa138 2 weeks ago /bin/sh -c #(nop) CMD ["/bin/bash"] 0B 
     2 weeks ago /bin/sh -c mkdir -p /run/systemd && echo 'do… 7B 
     2 weeks ago /bin/sh -c sed -i 's/^#\s*\(deb.*universe\)$… 2.76kB 
     2 weeks ago /bin/sh -c rm -rf /var/lib/apt/lists/* 0B 
     2 weeks ago /bin/sh -c set -xe && echo '#!/bin/sh' > /… 745B 
     2 weeks ago /bin/sh -c #(nop) ADD file:28c0771e44ff530db… 81.1MB

Each layer has an unique id associated with it and will be cached locally.  The reason we see the tag in the above output is because those layers are not locally present but exists in the DockerHub repository.

If we try to rebuild the same image and give it a new version, we'll see that Docker builds the new image using various layers of the image already present in the local cache.

    $ docker image build -t muchfuntolearn:v1.0 .
    Sending build context to Docker daemon 17.92kB
    Step 1/4 : FROM ubuntu
     ---> 113a43faa138
    Step 2/4 : COPY ./testfile /var/local
     ---> Using cache
     ---> f6500304a128
    Step 3/4 : WORKDIR /var/local
     ---> Using cache
     ---> cf189f932f95
    Step 4/4 : CMD ["cat", "testfile"]
     ---> Using cache
     ---> 912fe16e4752
    Successfully built 912fe16e4752
    Successfully tagged muchfuntolearn:v1.0

Suppose, we would like to modify the contents of the *testfile*, just like any real world application that gets changed frequently, and rebuild a new image version, we follow the same steps.

    $ echo "I want to learn more about Docker containers" >> testfile
    $ docker image build -t muchfuntolearn:v2.0 .
    Sending build context to Docker daemon 17.92kB
    Step 1/4 : FROM ubuntu
     ---> 113a43faa138
    Step 2/4 : COPY ./testfile /var/local
     ---> f6500304a128
    Step 3/4 : WORKDIR /var/local
    Removing intermediate container 0bf6987cef00
     ---> cf189f932f95
    Step 4/4 : CMD ["cat", "testfile"]
     ---> Running in 3b958d65350d
    Removing intermediate container 3b958d65350d
     ---> 912fe16e4752
    Successfully built 912fe16e4752
    Successfully tagged muchfuntolearn:v2.0
    $
    $ docker container run muchfuntolearn:v2.0
    It's SO MUCH FUN to learn Docker containers
    I want to learn more about Docker containers

## 

## Wrap it up:

Finally, let's wrap up this exercise by creating a docker image for a simple real world web application.

Here is a simple nodejs hello-world app that we'll use.  All it does is prints "Hello World!" on the webpage.

    $ cat app.js 
    var http = require('http');

    //create a server object:
    http.createServer(function (req, res) {
     res.writeHead(200, {'Content-Type': 'text/html'});
     res.write('Hello World!'); //write a response to the client
     res.end(); //end the response
    }).listen(3000); //the server object listens on port 8080
    $

Here is the Dockerfile we'll use to build the app container image:

    $ cat Dockerfile 
    FROM alpine:latest
    RUN apk update && apk add nodejs
    RUN mkdir -p /usr/src/app
    COPY . /usr/src/app
    WORKDIR /usr/src/app
    EXPOSE 3000 
    CMD ["node","app.js"]

The Dockerfile tells Docker to install nodejs on an alpine base container image and then copy the app from the host directory to the container image work directory /usr/src/app and finally ask it to run the command "node app.js".  The EXPOSE command tells Docker to allow traffic to port 3000.

Let's build the version 0.1 of the *hello-world* container image now:

    $ docker image build -t hello-world:v0.1 .
    Sending build context to Docker daemon 17.92kB
    Step 1/7 : FROM alpine:latest
     ---> 3fd9065eaf02
    Step 2/7 : RUN apk update && apk add nodejs
     ---> Using cache
     ---> 928e995e38f4
    Step 3/7 : RUN mkdir -p /usr/src/app
     ---> Using cache
     ---> 5c3b1d4fa148
    Step 4/7 : COPY . /usr/src/app
     ---> Using cache
     ---> dab04cdb33d6
    Step 5/7 : WORKDIR /usr/src/app
     ---> Using cache
     ---> c3fdc62b6f66
    Step 6/7 : EXPOSE 3000
     ---> Using cache
     ---> dbe68495bcca
    Step 7/7 : CMD ["node","app.js"]
     ---> Using cache
     ---> 74a062a6a3c3
    Successfully built 74a062a6a3c3
    Successfully tagged hello-world:v0.1

Use the following command to run the *hello-world* container app.  The command requests Docker to run the container in the background as a daemon \[-d\] and map the host port \[8000\] to the docker port \[3000\].  The nodejs HTTP server app will listen on TCP port 3000 in the container's network namespace.

    $ docker run -d -p 8000:3000 hello-world:v0.1 
    b6bbc0790c69ee95d9e977caef329f5b166c28de9f883614769058b781419bc7

The option \[-p\] in the above command is the port mapping option.  It requests Docker daemon to add a port NAT entry in the host's NAT table as shown below:

    $ sudo iptables -t nat --list 
    Chain PREROUTING (policy ACCEPT)
    target prot opt source destination 
    DOCKER all -- anywhere anywhere ADDRTYPE match dst-type LOCAL

    Chain INPUT (policy ACCEPT)
    target prot opt source destination

    Chain OUTPUT (policy ACCEPT)
    target prot opt source destination 
    DOCKER all -- anywhere !localhost/8 ADDRTYPE match dst-type LOCAL

    Chain POSTROUTING (policy ACCEPT)
    target prot opt source destination 
    MASQUERADE all -- 172.17.0.0/16 anywhere 
    MASQUERADE tcp -- 172.17.0.2 172.17.0.2 tcp dpt:3000

    Chain DOCKER (2 references)
    target prot opt source destination 
    RETURN all -- anywhere anywhere 
    DNAT tcp -- anywhere anywhere tcp dpt:8000 to:172.17.0.2:3000

Check if the container is running on the specified port.

    $ docker container ls
    CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
    b6bbc0790c69 hello-world:v0.1 "node app.js" 2 minutes ago Up 2 minutes 0.0.0.0:8000->3000/tcp agitated_zhukovsky

Now if we open a browser in the host machine and point it to the url: http://localhost:8000 it will print the message "Hello World!"

![Hello-World](https://ganeshtechblog.files.wordpress.com/2018/06/hello-world1.png){.alignnone .size-full .wp-image-903 width="2564" height="926"}

That's all folks !  Wasn't it so much fun to learn how to build Docker images? Pat yourselves on the back!  You have completed part-2 of the Docker container tutorial.

 
