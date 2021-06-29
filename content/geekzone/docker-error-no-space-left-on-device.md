---
title: Docker Error No Space Left On Device
description: ""
author: Ganesh Velrajan
tags: [
    GeekZone, Docker, Docker Compose
]
date: 2021-06-25
categories: [
    GeekZone, Docker
]
images: ["/images/docker/docker.png"]
---

Recently, one of my cloud native applications running as a Docker containers failed to function correctly all of a sudden.   It turned out that the problem was due to lack of disk space.  Docket daemon in specific is the one that complained about the lack of disk space in the host machine.  Here is the Docker error that complained about no space left on device:

``` {.source-code}
docker: Error response from daemon: no space left on device.
```

In this article I'll discuss how I peformed RCA (Root Cause Analysis) for this problem and what solutions I used to completely address the issue once and for all.

## Problem scenario in detail
My application has several microservices, each running as a Docker container in a VM(Host).  The front end microservice was the one that failed to function correctly.  Upon investigation I found out that the API requests made by the front end microservice was responded by the backend's API gateway with empty responses.

Upon further investigating the backend microservice, it appeared that all SQL queries to the postgres DB was failing.

I tried restarting the postgres DB docker container but the problem still persisted with a new error.

``` {.source-code}
docker: Error response from daemon: mkdir /var/lib/docker/overlay2/5051a8704320241cb6e3b92d5911b0769f2c93fbe02a99d33558c95861a64784-init: no space left on device.
```
I tried the `docker system prune` command to delete any unwanted images, stale containers and stale networks that are currently not used in the production system. This will free up some disk space for the running docker containers to use.  But this solution solved my problem for a day or two and the problem re-appeared.

So this time, I decided I'll do a complete deep dive to understand the real root cause of the problem - no space left on disk - and resolve it.

## Root Cause Analysis
I initially looked at the logs from each of the microservices chain to understand where the source of the problem is.  In my case, the backend microservice failed to connect with the DB microservice to perform SQL queries.  This was evident from the backend microservice logs.

Now I know that the docker daemon is complaining about creating a new file or write to an already existing file in its directory.  Usually docker filesystem is under /var/lib/docker in Linux.

### Check where the /var/lib/docker is mounted
The disk free (df) utility command comes in handy to find out the disk where the `/var/lib/docker`folder is mounted on; it tells which disk is full and by how much.

``` {.source-code}
$ df -h /var/lib/docker
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        49G   42G  6.4G  87% /
```
It says, `/var/lib/docker` is mounted on `/dev/root` filesystem.  Moreover, it also says that `/dev/root` filesystem is almost 87% full.  This is a clear indication that the `/dev/root` could have went out of space in the recent past and that could have prevented the docker daemon from writing any new data to a file on disk or create a new file in disk.

### Check the free inodes available
Check if the docker error is occurring due to lack of inodes availability.

``` {.source-code}
$ df -hi /var/lib/docker
Filesystem     Inodes IUsed IFree IUse% Mounted on
/dev/root        6.2M  226K  6.0M    4% /
```
It says, `/dev/root` filesystem uses only 4% or 226K inodes out of the available inodes 100% or 6.2M inodes.  So clearly, inode availability is not the cause of the problem here.  Free inodes are available in plenty for the docker daemon to create any new file under `/var/lib/docker`

### Check the disk usage of the docker directory
The `df` (disk free) command used above tells us about the disk free and disk usage information at the file system level only.  It doesn't tell which directory (or application) using the `/dev/root` disk filesystem is consuming the most data on the disk.  For that we need to use the `du` (disk usage) command.

In most cases, the application complaining the error ["Error: no space left on device"], is the one that consumes a lot of space on disk.  So first let's check the disk usage of `/var/lib/docker` directory in the `/dev/root` filesystem.

``` {.source-code}
$ sudo du -h /var/lib/docker
4.0K	/var/lib/docker/tmp
4.0K	/var/lib/docker/overlay2/472e0a836ebf11b07eb16761815326dff8861abfaf3d99de31c1efe2c7414bf3/work
4.0K	/var/lib/docker/overlay2/472e0a836ebf11b07eb16761815326dff8861abfaf3d99de31c1efe2c7414bf3/diff/var/local/webserver
8.0K	/var/lib/docker/overlay2/472e0a836ebf11b07eb16761815326dff8861abfaf3d99de31c1efe2c7414bf3/diff/var/local
12K	/var/lib/docker/overlay2/472e0a836ebf11b07eb16761815326dff8861abfaf3d99de31c1efe2c7414bf3/diff/var
16K	/var/lib/docker/overlay2/472e0a836ebf11b07eb16761815326dff8861abfaf3d99de31c1efe2c7414bf3/diff
32K	/var/lib/docker/overlay2/472e0a836ebf11b07eb16761815326dff8861abfaf3d99de31c1efe2c7414bf3
4.0K	/var/lib/docker/overlay2/f0475e9472279658f679d8980b1776f2c00aedc6ccfd6e31a02dcbebe3d3a2df/work
12K	/var/lib/docker/overlay2/f0475e9472279658f679d8980b1776f2c00aedc6ccfd6e31a02dcbebe3d3a2df/diff/var/lib/dpkg
16K	/var/lib/docker/overlay2/f0475e9472279658f679d8980b1776f2c00aedc6ccfd6e31a02dcbebe3d3a2df/diff/var/lib
20K	/var/lib/docker/overlay2/f0475e9472279658f679d8980b1776f2c00aedc6ccfd6e31a02dcbebe3d3a2df/diff/var
4.0K	/var/lib/docker/overlay2/f0475e9472279658f679d8980b1776f2c00aedc6ccfd6e31a02dcbebe3d3a2df/diff/usr/share/postgresql/12
64K	/var/lib/docker/overlay2/f0475e9472279658f679d8980b1776f2c00aedc6ccfd6e31a02dcbebe3d3a2df/diff/usr/share/postgresql
68K	/var/lib/docker/overlay2/f0475e9472279658f679d8980b1776f2c00aedc6ccfd6e31a02dcbebe3d3a2df/diff/usr/share
72K	/var/lib/docker/overlay2/f0475e9472279658f679d8980b1776f2c00aedc6ccfd6e31a02dcbebe3d3a2df/diff/usr
...
...
...
4.0K	/var/lib/docker/containers/63955d57625448a53e4a391dcf6f97952e30f44a13ee65ac24d90e747a1ffca1/checkpoints
4.0K	/var/lib/docker/containers/63955d57625448a53e4a391dcf6f97952e30f44a13ee65ac24d90e747a1ffca1/mounts
36K	/var/lib/docker/containers/63955d57625448a53e4a391dcf6f97952e30f44a13ee65ac24d90e747a1ffca1
4.0K	/var/lib/docker/containers/23a77fd2325f88deec8cb45480326a1044e914b08b85553ae9d0d29b19e81698/checkpoints
4.0K	/var/lib/docker/containers/23a77fd2325f88deec8cb45480326a1044e914b08b85553ae9d0d29b19e81698/mounts
7.0M	/var/lib/docker/containers/23a77fd2325f88deec8cb45480326a1044e914b08b85553ae9d0d29b19e81698
4.0K	/var/lib/docker/containers/14f5a4c98e5653a15e334880b9a58504adda416b06f359655bf251073b160ab1/checkpoints
4.0K	/var/lib/docker/containers/14f5a4c98e5653a15e334880b9a58504adda416b06f359655bf251073b160ab1/mounts
35G	/var/lib/docker/containers/14f5a4c98e5653a15e334880b9a58504adda416b06f359655bf251073b160ab1
4.0K	/var/lib/docker/containers/3d03387e4de11ca77ce0dc4399f8bb58f62cd70faabbe36b7230355dcbc6b39b/checkpoints
4.0K	/var/lib/docker/containers/3d03387e4de11ca77ce0dc4399f8bb58f62cd70faabbe36b7230355dcbc6b39b/mounts
42M	/var/lib/docker/containers/3d03387e4de11ca77ce0dc4399f8bb58f62cd70faabbe36b7230355dcbc6b39b
35G	/var/lib/docker/containers
4.0K	/var/lib/docker/swarm
4.0K	/var/lib/docker/buildkit/executor
4.0K	/var/lib/docker/buildkit/content/ingest
8.0K	/var/lib/docker/buildkit/content
92K	/var/lib/docker/buildkit
16K	/var/lib/docker/plugins
56K	/var/lib/docker/network/files
60K	/var/lib/docker/network
38G	/var/lib/docker
```
If you look at the output above, it clearly shows that `/var/lib/docker` directory alone consumes 38G out of the available 50GB on the `/dev/root` file system.  Also it specifically says that docker container ID `14f5a4c98e5653a15e334880b9a58504adda416b06f359655bf251073b160ab1` consumes 35G of disk space.

``` {.source-code}
35G	/var/lib/docker/containers/14f5a4c98e5653a15e334880b9a58504adda416b06f359655bf251073b160ab1
```

I checked if my application container writes anything to the container filesystem.  It appeared that my application wrote only less than 1MB of data to to a docker volume.  Also it wrote only to the docker volume under `/var/lib/docker/volumes`and not to the container file system `/var/lib/docker/containers`.  So it doesn't explain why the container file system is so bulky (35GB).

I finally checked the log file where the container logs are stored.  

``` {.source-code}
$ sudo sh -c "du -ch /var/lib/docker/containers/*/*-json.log"
35G	/var/lib/docker/containers/14f5a4c98e5653a15e334880b9a58504adda416b06f359655bf251073b160ab1/14f5a4c98e5653a15e334880b9a58504adda416b06f359655bf251073b160ab1-json.log
7.0M	/var/lib/docker/containers/23a77fd2325f88deec8cb45480326a1044e914b08b85553ae9d0d29b19e81698/23a77fd2325f88deec8cb45480326a1044e914b08b85553ae9d0d29b19e81698-json.log
42M	/var/lib/docker/containers/3d03387e4de11ca77ce0dc4399f8bb58f62cd70faabbe36b7230355dcbc6b39b/3d03387e4de11ca77ce0dc4399f8bb58f62cd70faabbe36b7230355dcbc6b39b-json.log
4.0K	/var/lib/docker/containers/63955d57625448a53e4a391dcf6f97952e30f44a13ee65ac24d90e747a1ffca1/63955d57625448a53e4a391dcf6f97952e30f44a13ee65ac24d90e747a1ffca1-json.log
35G	total
```

## Root Cause of the Problem
The above output shows that the JSON log file of container `14f5a4c98e5653a15e334880b9a58504adda416b06f359655bf251073b160ab1` is 35GB in size.

This is the root cause of the problem.  The container or the application is generating too many logs or the docker container has been running for several months without being restarted or respawned.

In my case, both were causing the problem.  My container has been running for more than 2 months undisturbed and also it was logging all the Postgres DB SQL queries to stdout.  

Mainly, I didn't explicitly configure the docker daemon to rotate the log file when it reaches a specific size, say 200MB or 1GB on disk.

## How to solve the issue
There is a temporary solution and a permanent solution to this problem.

### Temporary Solution
The temporary solution is to free up the disk space to prevent the application from misbehaving or crashing in a production system.  This is a more easy and quick solution than killing and restarting the container in a production system.

You could use either one of the commands below to truncate the log file.  You need to be a sudo user to write to the log file.  I highly recommend using the truncate command and it worked well for me.

``` {.source-code}
echo "" > $(docker inspect --format='{{.LogPath}}' <container_name_or_id>)
```

``` {.source-code}
truncate -s 0 $(docker inspect --format='{{.LogPath}}' <container_name_or_id>)
```

**Note:**  Do this at your own risk, as you are trying to compete with the docker daemon to write into the log file that has been opened and used by docker daemon.

After you truncate the log file, make sure that `docker logs <container-id>` shows the newer logs.  If there was any mess up due to the truncate command, you'll get an error like the one shown below:

``` {.source-code}
error from daemon in stream: Error grabbing logs: invalid character '\x00' looking for beginning of value
```
This error means you have messed up the json contents of the JSON log file.  It will do no harm to the functionality of running container but just that you'll no longer be able to see meaning logs from the `docker logs` command output.

But, when you remove and recreate the container to implement the permanent solution explained in the next section (during a scheduled maintenance window of your production system), this problem would go away.

### Permanent Solution
The permanent solution is to configure docker daemon or the docker container to use only the maximum specified size for the log file.  When the max file size is reached, the docker daemon should rotate the logs and start logging from the beginning of the file.  This way the log file doesn't grow in size automatically.

Also, I had to disable logging in my GORM postgres DB driver to prevent unnecessary logging for each SQL query in a production build.

Follow the instructions in the docker documentation page on [JSON File logging driver](https://docs.docker.com/config/containers/logging/json-file/) for setting up the max log file size and to turn on the automatic log rotation feature.

Basically, you need to create a daemon.json file under /etc/docker directory with the following content.

``` {.source-code}
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3" 
  }
}
```
It basically limits the JSON log file size to 10MB maximum and thereafter the file will be rotated automatically and logged from the beggining of the file.

Also, the daemon could create maximum 3 such log files only.  The default value for this field is 1.

**Note:** You need to restart the docker daemon to make this configuration to take effect.  Also all the containers running in the host need to be restarted.

Alternatively, you could just configure a container to limit the log file size and enable automatic log rotation using the following command:

``` {.source-code}
$ docker run \
      --log-driver json-file --log-opt max-size=10m \
      --log-opt max-file=3 \
      alpine echo hello world
```

That's all folks!  Hope that was useful.