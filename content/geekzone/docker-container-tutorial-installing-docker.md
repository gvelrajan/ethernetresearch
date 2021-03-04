---
title: Docker Container Tutorial - Running A Simple Docker Container
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

## How to install Docker CE(Community Edition)

Docker CE is the mini version of Docker available to download for free for beginners to learn and experiment with Docker containers on a laptop.

For Linux  platform, you could install Docker CE by running a simple script provided by Docker.  Visit the official Docker website to find out how to install a version of Docker CE for your OS.

``` 
$ curl -fsSL get.docker.com -o get-docker.sh

$ sudo sh get-docker.sh
```

If you would like to use Docker as a non-root user, you should now consider adding your user to the “docker” group with something like:

``` 
sudo usermod -aG docker <your-user-name>
```

Remember to log out and back in for this to take effect!

``` 
WARNING: Adding a user to the "docker" group grants the ability to run

containers which can be used to obtain root privileges on the

docker host. Refer to https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface

for more information.
```

 

## Getting Started:

First check the version of docker installed on your OS:

``` 
$ docker --version

Docker version 18.05.0-ce, build f150324
```

Now check if we have any container images locally on our installation.

``` 
$ docker image ls

REPOSITORY TAG IMAGE ID CREATED SIZE
```

Now to run a basic “hello world” container using docker execute the following command:

    $ docker run hello-world

    Unable to find image 'hello-world:latest' locally

    latest: Pulling from library/hello-world

    9bb5a5d4561a: Pull complete

    Digest: sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77

    Status: Downloaded newer image for hello-world:latest

    Hello from Docker!

    This message shows that your installation appears to be working correctly.

    To generate this message, Docker took the following steps:

    1. The Docker client contacted the Docker daemon.

    2. The Docker daemon pulled the "hello-world" image from the Docker Hub.

    (amd64)

    3. The Docker daemon created a new container from that image which runs the

    executable that produces the output you are currently reading.

    4. The Docker daemon streamed that output to the Docker client, which sent it

    to your terminal.

    To try something more ambitious, you can run an Ubuntu container with:

    $ docker run -it ubuntu bash

    Share images, automate workflows, and more with a free Docker ID:

    https://hub.docker.com/

    For more examples and ideas, visit:

    https://docs.docker.com/engine/userguide/

    $

The output was verbose and self explanatory. Now check your docker images again.

``` 
$ docker image ls

REPOSITORY TAG IMAGE ID CREATED SIZE

hello-world latest e38bc07ac18e 2 months ago 1.85kB
```

Now check the status of the “hello world” container.

``` 
$ docker container ls

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES

```

There is no output because the “hello-world” container finished executing after printing the message. If you wanted to still see the container that has finished execution, include the “–all” option in the command.

``` 
$ docker container ls --all

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES

695c82c1b0f9 hello-world "/hello" 2 minutes ago Exited (0) 2 minutes ago jovial_bartik

```

Rerun the “hello-world” container again.

``` 
$ docker run hello-world

Hello from Docker!

This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:

1. The Docker client contacted the Docker daemon.

2. The Docker daemon pulled the "hello-world" image from the Docker Hub.

(amd64)

3. The Docker daemon created a new container from that image which runs the

executable that produces the output you are currently reading.

4. The Docker daemon streamed that output to the Docker client, which sent it

to your terminal.

To try something more ambitious, you can run an Ubuntu container with:

$ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:

https://hub.docker.com/

For more examples and ideas, visit:

https://docs.docker.com/engine/userguide/

$
```

If you carefully note, this time docker was able to find the “hello-world” image locally on our machine and didn’t try to pull it from the dockerhub again. The execution was much faster.

Now let’s try something more serious. Let’s run an Ubuntu container, interactively.  This time lets pull the image first and then run it using the run command.

    $ docker pull ubuntu
    Using default tag: latest
    latest: Pulling from library/ubuntu
    6b98dfc16071: Pull complete 
    4001a1209541: Pull complete 
    6319fc68c576: Pull complete 
    b24603670dc3: Pull complete 
    97f170c87c6f: Pull complete 
    Digest: sha256:5f4bdc3467537cbbe563e80db2c3ec95d548a9145d64453b06939c4592d67b6d
    Status: Downloaded newer image for ubuntu:latest

Check if we have the image in our local repo.

    $ docker image ls
    REPOSITORY TAG IMAGE ID CREATED SIZE
    ubuntu latest 113a43faa138 2 weeks ago 81.2MB
    hello-world latest e38bc07ac18e 2 months ago 1.85kB

Now run the container in interactive mode.

``` 
$ docker run -it ubuntu bash

root@1636ea923f40:/# pwd
/
root@1636ea923f40:/# ls
bin dev home lib64 mnt proc run srv tmp var
boot etc lib media opt root sbin sys usr
root@1636ea923f40:/# hostname
1636ea923f40
root@1636ea923f40:/# whoami
root
root@1636ea923f40:/# >testfile
root@1636ea923f40:/# ls
bin dev home lib64 mnt proc run srv testfile usr
boot etc lib media opt root sbin sys tmp var
root@1636ea923f40:/# exit
exit
$
```

As expected, docker didn’t find the ubuntu container image locally and hence it downloaded the image from the dockerhub. It started the container, executed the bash command and provided us the access to the shell. You can see that the shell prompt changed from $ to # when inside the container. Docker provided the container a unique hostname “1636ea923f40”.

When we exited the bash shell, the container finished its execution.

``` 
$ docker container ls --all

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES

1636ea923f40 ubuntu "bash" 8 minutes ago Exited (0) 8 minutes ago cocky_stallman

167b2e58aa67 hello-world "/hello" 21 minutes ago Exited (0) 21 minutes ago vigorous_euler

695c82c1b0f9 hello-world "/hello" 37 minutes ago Exited (0) 37 minutes ago jovial_bartik
```

From the output of the above command we can see that the hostname of the ubuntu container is same as its container id.

``` 
$ docker image ls
REPOSITORY TAG IMAGE ID CREATED SIZE
ubuntu latest 113a43faa138 2 weeks ago 81.2MB
hello-world latest e38bc07ac18e 2 months ago 1.85kB
```

If you noted carefully, I did something notorious inside the ubuntu container. I created a file named “testfile” in the root directory before I exited from it. This is just to see what happens if I restart the stopped container.

``` 
$ docker container start 1636 -i
root@1636ea923f40:/# pwd
/
root@1636ea923f40:/# ls
bin dev home lib64 mnt proc run srv testfile usr
boot etc lib media opt root sbin sys tmp var
root@1636ea923f40:/# exit
exit
$
```

It is enough to just specify the unique first few digits(1636 in this case) of a container id in the command line. We don’t have to specify the complete container id(1636ea923f40).

To our surprise, the “testfile” still exists in the root directory of the container. What does this tell us ? When a container finishes its execution, its root file system is not cleaned and removed. It gets removed only when the user makes an explicit request to remove it, by executing “docker container rm 1636”. 

Now let’s try to create one more ubuntu container, using the ubuntu image that was downloaded earlier.

``` 
$ docker run -it ubuntu bash
root@1affae2c7305:/# pwd
/
root@1affae2c7305:/# ls
bin dev home lib64 mnt proc run srv tmp var
boot etc lib media opt root sbin sys usr
root@1affae2c7305:/# >magicfile
root@1affae2c7305:/# ls
bin dev home lib64 media opt root sbin sys usr
boot etc lib magicfile mnt proc run srv tmp var
root@1affae2c7305:/# hostname
1affae2c7305
root@1affae2c7305:/# whoami
root
root@1affae2c7305:/# exit
exit
$
$ docker container ls --all
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
1affae2c7305 ubuntu "bash" About a minute ago Exited (0) About a minute ago friendly_davinci
1636ea923f40 ubuntu "bash" 34 minutes ago Exited (0) 11 minutes ago cocky_stallman
167b2e58aa67 hello-world "/hello" About an hour ago Exited (0) About an hour ago vigorous_euler
695c82c1b0f9 hello-world "/hello" About an hour ago Exited (0) About an hour ago jovial_bartik
```

I created a new ubuntu container and docker assigned it a new id, which is 1affae2c7305. As we expected docker created a new root file system for the container and hence we didn’t see the “testfile” that we created in the first Ubuntu container’s root file system. Now, I created a “magicfile” under the root directory.

This is just to prove the point that docker creates a new root file system when we ask it to create a new container. The file system is saved and not destroyed when the container exits. We could restart the container using the root file system whenever we want.

``` 
$ docker container start 1aff -i

root@1affae2c7305:/# pwd
/
root@1affae2c7305:/# ls
bin dev home lib64 media opt root sbin sys usr
boot etc lib magicfile mnt proc run srv tmp var
root@1affae2c7305:/# hostname
1affae2c7305
root@1affae2c7305:/# exit
exit
$
```

Now let’s remove the container with id 1affae2c7305.

``` 
$ docker container rm 1aff
1aff
$ docker container ls --all | grep ubuntu
1636ea923f40 ubuntu "bash" About an hour ago Exited (0) 28 minutes ago cocky_stallman
```

If we try to start the container now, it will fail.

``` 
$ docker container start 1aff -i
Error: No such container: 1aff
```

Now, let’s try removing the docker image present locally in our machine.

    $ docker image rm ubuntu
    Error response from daemon: conflict: unable to remove repository reference "ubuntu" (must force) - container 1636ea923f40 is using its referenced image 113a43faa138

It throws an error saying container 1636 is still referencing the image.  We should delete all the containers spawned using the ubuntu image before we could delete it.  So let’s delete the container 1636 also.

    $ docker container rm 1636
    1636
    $ docker container ls --all | grep ubuntu
    $ 
    $ docker image rm ubuntu
    Untagged: ubuntu:latest
    Untagged: ubuntu@sha256:5f4bdc3467537cbbe563e80db2c3ec95d548a9145d64453b06939c4592d67b6d
    Deleted: sha256:113a43faa1382a7404681f1b9af2f0d70b182c569aab71db497e33fa59ed87e6
    Deleted: sha256:a9fa410a3f1704cd9061a802b6ca6e50a0df183cb10644a3ec4cac9f6421677a
    Deleted: sha256:b21f75f60422609fa79f241bf80044e6e133dd0662851afb12dacd22d199233a
    Deleted: sha256:038d2d2aa4fb988c06f04e3af208cc0c1dbd9703aa04905ade206d783e7bc06a
    Deleted: sha256:b904d425ea85240d6af5a6c6f145e05d5e0127f547f8eb4f68552962df846e81
    Deleted: sha256:db9476e6d963ed2b6042abef1c354223148cdcdbd6c7416c71a019ebcaea0edb
    $ docker image ls
    REPOSITORY TAG IMAGE ID CREATED SIZE
    hello-world latest e38bc07ac18e 2 months ago 1.85kB
    $

Now the ubuntu image is successfully removed from our local repository.

That’s all the basic Docker container stuff you should be aware of folks.

Congratulations! You have completed the basic Docker tutorial.

 
