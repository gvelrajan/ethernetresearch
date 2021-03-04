---
title: Docker Containers Tutorial - Persistent Storage Volumes and Stateful Containers
description: ""
author: Ganesh Velrajan
tags: [
    GeekZone, Docker, Kubernetes, Persistent Storage Volume
]
date: 2018-10-23
categories: [
    GeekZone, Docker
]
images: ["/images/docker/docker.png"]
---

In this article, I'll show you how to mount persistent storage volumes inside a Docker container, so that Stateful Applications such as MySQL or MongoDB or PostgreSQL could be run as Docker Containers. We'll use MySQL as an example here.

## Why use Persistent Storage Volumes?

Containers are ephemeral in nature.  They have a filesystem of their own.  When containers die, the data stored locally in their filesystem is also gone.  Stateful applications such as MySQL or MongoDB or PostgreSQL cannot be run as Docker Containers, because the data stored in their database will be lost when the container crashes or dies or gets deleted.  Data is critical.  We don't want to lose it at any cost.

## What is a Persistent Storage Volume ?

A storage device or volume that can persist a container crash or its life cycle is called a Persistent Storage Volume.   Persistent storage volumes can be created in a disk mounted to the Host Machine directly or it could be created in a NAS storage device in the Local LAN network mounted as a network storage device on the Host Machine or it could even be created in a a cloud storage device mounted as a storage device in the Host Machine.

## How to create Persistent Storage Volumes - Two Methods:

There are two different methods to mount a persistent storage volume inside a Docker Container.

**Method \#1:**

You can create a new persistent storage volume in the Host Machine and mount it under a directory or folder inside a Docker Container.  The Docker Container gets exclusive access to the storage volume.   The data stored in the volume cannot be easily read, manipulated or corrupted from the Host Machine ( although, we could read the data through some means, I'll show you how, later in this article).

**Method \#2:**

You can mount a local directory in the host machine as a persistent storage volume inside a Docker Container, so that data could be shared between the host machine and Docker Container.  This method is very useful if the host machine wants to access or periodically backup the data or database written to the  folder by the DB server running inside the Docker Container.

**Demo:**

### Method \#1

Check if there are any existing volumes:

    $ docker volume ls

    DRIVER VOLUME NAME

    $

Create a new persistent storage volume in the Host Machine

    $ docker volume create mysql-data

    mysql-data

    $ docker volume ls

    DRIVER VOLUME NAME

    local mysql-data

    $

Inspect the storage volume to get more detailed information.

    $ docker volume inspect mysql-data
    [
       {
          "CreatedAt": "2018-10-23T03:18:12Z",
          "Driver": "local",
          "Labels": {},
          "Mountpoint": "/var/lib/docker/volumes/mysql-data/_data", 
          "Name": "mysql-data",
          "Options": {},
          "Scope": "local"
       }
    ]

Check the data in the storage volume, from where it is mounted locally.  You need root privileges to access the folder where it is mounted in the host machine.

    $ ls /var/lib/docker/volumes/mysql-data/_data
    ls: cannot access '/var/lib/docker/volumes/mysql-data/_data': 
    Permission denied
    $ sudo ls /var/lib/docker/volumes/mysql-data/_data
    $

It is an empty volume with no data, as we expected.

Create a MySQL DB  (stateful) Docker Container and make it use the persistent storage volume we just created.  So that, MySQL will store its DB and files in this volume.  MySQL stores its data at the /var/lib/mysql folder.

    $ docker run --name ganesh-mysql -v mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=mypasswd -d mysql:latest

    Unable to find image 'mysql:latest' locally
    latest: Pulling from library/mysql
    f17d81b4b692: Pull complete 
    c691115e6ae9: Pull complete 
    41544cb19235: Pull complete 
    254d04f5f66d: Pull complete 
    4fe240edfdc9: Pull complete 
    0cd4fcc94b67: Pull complete 
    8df36ec4b34a: Pull complete 
    739800af3a9f: Pull complete 
    0cbc995daddd: Pull complete 
    5db5c83b9b9a: Pull complete 
    9cb56d3f0a7e: Pull complete 
    448d4de73cac: Pull complete 
    Digest: sha256:8fdc47e9ccb8112a62148032ae70484e3453b628ab6fe02bccf159e2966b
    750e
    Status: Downloaded newer image for mysql:latest
    e15c04f42ec6e5b0e0ae2256caf0f6ed4760c8f27806f300f9246bd0718dbe37
    $

Now, get into the MySQL Docker Container's bash shell and check the /var/lib/mysql folder.

    $ docker exec -it e15c /bin/bash
    root@e15c04f42ec6:/# ls
    bin docker-entrypoint-initdb.d home media proc sbin tmp
    boot entrypoint.sh lib mnt root srv usr
    dev etc lib64 opt run sys var

    root@e15c04f42ec6:/# ls /var/lib/mysql
    auto.cnf ca.pem ib_logfile1 performance_schema sys
    binlog.000001 client-cert.pem ibdata1 private_key.pem undo_001
    binlog.000002 client-key.pem ibtmp1 public_key.pem undo_002
    binlog.index ib_buffer_pool mysql server-cert.pem
    ca-key.pem ib_logfile0 mysql.ibd server-key.pem

    root@e15c04f42ec6:/# exit
    exit
    $

The folder has the mysql database and files.

Now let's check the same from the host machine's mount point folder.

    $ sudo ls /var/lib/docker/volumes/mysql-data/_data
    auto.cnf ca.pem ib_logfile0 performance_schema sys
    binlog.000001 client-cert.pem ib_logfile1 private_key.pem undo_001
    binlog.000002 client-key.pem ibtmp1 public_key.pem undo_002
    binlog.index ib_buffer_pool mysql server-cert.pem
    ca-key.pem ibdata1 mysql.ibd server-key.pem
    $

They reflect the same information.  This is because it's a volume created in the host namespace and mounted inside the container.

Now, if we kill and create a new MySQL docker container, it can reuse the MySQL database created by the previous incarnation, as it is.  That's the power of persistent storage volumes.  They persist a container crash or death.

    $ docker ps
    CONTAINER ID IMAGE COMMAND CREATED 
    STATUS PORTS NAMES
    e15c04f42ec6 mysql:latest "docker-entrypoint.s…" 10 minutes
    ago Up 10 minutes 3306/tcp, 33060/tcp ganesh-mysql

    $ docker container stop ganesh-mysql
    ganesh-mysql
    $ docker container rm ganesh-mysql
    ganesh-mysql

    $ docker container ps -a
    CONTAINER ID IMAGE COMMAND CREATED 
    STATUS PORTS NAMES
    $

Let's get the timestamp and snapshot of the files in the mysql-data storage volume we created.

    $ sudo ls -l /var/lib/docker/volumes/mysql-data/_data
    total 179200
    -rw-r----- 1 999 docker 56 Oct 23 03:31 auto.cnf
    -rw-r----- 1 999 docker 3071644 Oct 23 03:31 binlog.000001
    -rw-r----- 1 999 docker 178 Oct 23 03:42 binlog.000002
    -rw-r----- 1 999 docker 155 Oct 23 03:44 binlog.000003
    -rw-r----- 1 999 docker 48 Oct 23 03:44 binlog.index
    -rw------- 1 999 docker 1680 Oct 23 03:31 ca-key.pem
    -rw-r--r-- 1 999 docker 1112 Oct 23 03:31 ca.pem
    -rw-r--r-- 1 999 docker 1112 Oct 23 03:31 client-cert.pem
    -rw------- 1 999 docker 1680 Oct 23 03:31 client-key.pem
    -rw-r----- 1 999 docker 4953 Oct 23 03:42 ib_buffer_pool
    -rw-r----- 1 999 docker 12582912 Oct 23 03:44 ibdata1
    -rw-r----- 1 999 docker 50331648 Oct 23 03:44 ib_logfile0
    -rw-r----- 1 999 docker 50331648 Oct 23 03:30 ib_logfile1
    -rw-r----- 1 999 docker 12582912 Oct 23 03:44 ibtmp1
    drwxr-x--- 2 999 docker 4096 Oct 23 03:31 mysql
    -rw-r----- 1 999 docker 31457280 Oct 23 03:44 mysql.ibd
    drwxr-x--- 2 999 docker 4096 Oct 23 03:31 performance_schema
    -rw------- 1 999 docker 1676 Oct 23 03:31 private_key.pem
    -rw-r--r-- 1 999 docker 452 Oct 23 03:31 public_key.pem
    -rw-r--r-- 1 999 docker 1112 Oct 23 03:31 server-cert.pem
    -rw------- 1 999 docker 1680 Oct 23 03:31 server-key.pem
    drwxr-x--- 2 999 docker 4096 Oct 23 03:31 sys
    -rw-r----- 1 999 docker 12582912 Oct 23 03:44 undo_001
    -rw-r----- 1 999 docker 10485760 Oct 23 03:44 undo_002

Now create a new MySQL Docker Container and make it use the same storage volume where MySQL DB and files already exists.

    $ docker run --name ganesh-mysql-v2 -v mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=mypasswd -d mysql:latest
    46973aa2cddb3b65c94ec37bdee3e8b044767aa4ccc0962900e712eefd4710ec
    $

    $ sudo ls -l /var/lib/docker/volumes/mysql-data/_data
    total 179200
    -rw-r----- 1 999 docker 56 Oct 23 03:31 auto.cnf
    -rw-r----- 1 999 docker 3071644 Oct 23 03:31 binlog.000001
    -rw-r----- 1 999 docker 178 Oct 23 03:42 binlog.000002
    -rw-r----- 1 999 docker 155 Oct 23 03:44 binlog.000003
    -rw-r----- 1 999 docker 48 Oct 23 03:44 binlog.index
    -rw------- 1 999 docker 1680 Oct 23 03:31 ca-key.pem
    -rw-r--r-- 1 999 docker 1112 Oct 23 03:31 ca.pem
    -rw-r--r-- 1 999 docker 1112 Oct 23 03:31 client-cert.pem
    -rw------- 1 999 docker 1680 Oct 23 03:31 client-key.pem
    -rw-r----- 1 999 docker 4953 Oct 23 03:42 ib_buffer_pool
    -rw-r----- 1 999 docker 12582912 Oct 23 03:44 ibdata1
    -rw-r----- 1 999 docker 50331648 Oct 23 03:44 ib_logfile0
    -rw-r----- 1 999 docker 50331648 Oct 23 03:30 ib_logfile1
    -rw-r----- 1 999 docker 12582912 Oct 23 03:44 ibtmp1
    drwxr-x--- 2 999 docker 4096 Oct 23 03:31 mysql
    -rw-r----- 1 999 docker 31457280 Oct 23 03:44 mysql.ibd
    drwxr-x--- 2 999 docker 4096 Oct 23 03:31 performance_schema
    -rw------- 1 999 docker 1676 Oct 23 03:31 private_key.pem
    -rw-r--r-- 1 999 docker 452 Oct 23 03:31 public_key.pem
    -rw-r--r-- 1 999 docker 1112 Oct 23 03:31 server-cert.pem
    -rw------- 1 999 docker 1680 Oct 23 03:31 server-key.pem
    drwxr-x--- 2 999 docker 4096 Oct 23 03:31 sys
    -rw-r----- 1 999 docker 12582912 Oct 23 03:44 undo_001
    -rw-r----- 1 999 docker 10485760 Oct 23 03:44 undo_002

We see that the files and their timestamp has not changed.  This proves that the new MySQL instances reused the database and files at /var/lib/mysql.  It also proves that the data is persistent across a container crash.

 

### Method \#2:

First, let's create a directory in the host machine.

    $ mkdir mysql-data-dir
    $ ls mysql-data-dir/
    $

Now share the directory in the host-machine with the MySQL Docker Container.

    $ docker run --name ganesh-mysql-v3   
   -v /home/user/mysql-data-dir:/var/lib/mysql   
   -e MYSQL_ROOT_PASSWORD=mypasswd -d mysql:latest
    a809f1fa7608714641148a47889d53894a60e9ead2f21b177d411dc7197da21a

    $ ls /home/user/mysql-data-dir
    auto.cnf client-cert.pem ib_logfile1 private_key.pem undo_001
    binlog.000001 client-key.pem ibtmp1 public_key.pem undo_002
    binlog.index ib_buffer_pool mysql server-cert.pem
    ca-key.pem ibdata1 mysql.ibd server-key.pem
    ca.pem ib_logfile0 performance_schema sys
    $

We see that the mysql container has written its files in the folder we shared with it from the host machine.

That's all folks !

You can see this tutorial in action in these video lectures below.

### Tutorial:

https://youtu.be/rIxOLB2k-Xc

### Demo:

https://youtu.be/YTNpSxY_icw
