---
title: Kubernetes Microservice Remote Access
description: ""
author: Ganesh Velrajan
tags: [
    Container, Docker, Kubernetes, Microservice, Remote Access
]
date: 2020-10-29
categories: [
    GeekZone, Kubernetes
]
images: ["/images/kubernetes/kubernetes.jpg"]
---

In this document, we'll discuss how to develop, debug, and test a microservice locally on your laptop, without having to run all the other microservices of an app also on your laptop.

The solution discussed in this document could also be used to debug, fix, and test a problem reported by your customer in your live production cluster by routing accesses to your microservice (that has the bug) to the one running in your laptop with the fix.

## Problem Statement:
Let's say your team is focussed on developing and testing a front-end ReactJS microservice for your company's application that has like 25 other microservices such as Spring Boot, Elastic Search, Redis, Postgres, Grafana, and so on, in it.

As a dev, you would usually try to run all of these microservices in your laptop so that you could develop,build and test changes to your front-end ReactJS App natively on your laptop.  But running all these 25 microservices on your laptop will make it into a toaster, because you are pushing your laptop cpu, memory and power consumption to their maximum capacity throughout the day.   

## Workarounds Used Today
Many try to workaround this problem by upgrade their RAM or disk or fan module or even the laptop itself.  

This adds more costs to your organization. Sometimes it may make you wonder, "Was it really a smart decision to convert your monolithic applications into microservices, in the first place?"  

Many organizations that have started their journey into the microservices world, for the benefits it offers, have faced the exact same problem. And, fortunately, [SocketXP](https://www.socketxp.com) has a solution to address this problem.

## Wall Power Sockets Analogy
Let me ask you this question. Would you run a huge power generator in your house just because you needed some electricity to run your TV, Air Conditioner and Washing machine? The answer would be "no". You'd just install a couple of sockets on your wall, run a power cable to the energy company in your city and hook up your electrical equipments to those wall sockets. Neat and simple isn't it?

Then why do we try to run all our microservices(both 800 pound gorillas and tiny little mice ones) on that poor little laptop on your lap, all day long, just because you need to focus on developing and testing one front-end microservice?  You could very well install software "wall sockets" on your laptop and have your ReactJS front-end microservice talk to 25 other microservices running remotely in a cloud (or some on-prem cluster perhaps?) via the software "wall sockets" And vice-versa.

Our SocketXP Microservice Remote Access solution would help create those software "wall-sockets" in your laptop, for all your remote microservices.

## How SocketXP Microservice Remote Access Solution Works?
You need to download and run a SocketXP proxy agent (also available as Docker Container) on your laptop and on your remote cluster where other N-1 microservices of your application reside.

SocketXP proxy agent will create "TCP/IP" sockets on your laptop (one for each of the microservice running in a remote cluster) and run a bidirectional TLS tunnel between your laptop and your remote cluster.  We'd also adjust DNS records (for those remotely running microservices) in your laptop to route to the localhost IP address.  So that, any requests from your front-end microservices to these 25 other microservices, would be routed to these local sockets.

::: tip Why this is important:
Your front-end microservice would continue to connect to other microservices of the app as usual, without requiring any modifications to its code or configurations.
:::

## Docker Container Demo
For this exercise, we are going to run Postgres DB as a Docker Container in a remote server and make a python based backend microservice (under development in my laptop) access the remote DB via the SocketXP proxy. 

Postgres DB is accessible via the hostname `postgres` and port `5432`

Here is the simple backend microservice written in python.
``` python
$cat backend.py 
import psycopg2

con = psycopg2.connect(database="postgres", user="gvelrajan", password="pa$$word123", host="postgres", port="5432")

print("Database opened successfully")

cur = con.cursor()
cur.execute('''CREATE TABLE STUDENT
      (ADMISSION INT PRIMARY KEY     NOT NULL,
      NAME           TEXT    NOT NULL,
      AGE            INT     NOT NULL,
      COURSE        CHAR(50),
      DEPARTMENT        CHAR(50));''')

print("Table created successfully")

cur.execute("INSERT INTO STUDENT (ADMISSION,NAME,AGE,COURSE,DEPARTMENT) VALUES (34231, 'Dave', 19, 'Information Technology', 'ICT')");
cur.execute("INSERT INTO STUDENT (ADMISSION,NAME,AGE,COURSE,DEPARTMENT) VALUES (34232, 'Gary', 18, 'Electrical Engineering', 'ICT')");
cur.execute("INSERT INTO STUDENT (ADMISSION,NAME,AGE,COURSE,DEPARTMENT) VALUES (34233, 'Abel', 17, 'Computer Science', 'ICT')");

print("Student records added to the table successfully")

con.commit()
con.close()
```
The python backend microservice connects to the DB using the hostname 'postgres' and port '5432'

### Run the Postgres DB Docker Container

First create a docker volume
``` sh
$ docker volume create postgres
```
Next run the Postgres DB container and mount the `postgres` volume in the host machine to the directory `/var/lib/postgresql/data` inside the container.

``` sh
$ docker run --name postgres --restart unless-stopped --net appnet --mount source=postgres,target=/var/lib/postgresql/data -e POSTGRES_USER=gvelrajan -e POSTGRES_PASSWORD=pa$$word123 -e POSTGRES_DB=postgres -d postgres:latest
```
### Run SocketXP Proxy Agent container in a Cluster/Server
First go to [SocketXP Portal](https://portal.socketxp.com). Signup for a free account and get your authtoken there.

Use your authtoken in the below config file.
``` json
$ cat config.json
{
    "authtoken": "<your-auth-token-goes-here>"
    "tunnel_enabled": true, 
    "tunnels" : [{ 
        "destination": "tcp://postgres:5432", 
        "iot_device_id": "postgres-mservice-cluster1", 
    }] 
}
```
Store the config.json file in a local directory(/home/gvelrajan/config/config.json) and map the directory into the SocketXP docker container at /data as shown in the command below to start the docker container.

``` sh
docker run --name socketxp --restart unless-stopped --net appnet -d  -v /home/gvelrajan/config:/data expresssocket/socketxp:latest
```
Execute the following command to check the logs for success status or any errors:
``` sh
docker logs socketxp
```
It would have something like this:
``` sh
Connected.
TCP tunnel [gvelrajan-5bz4y6hi] created.
```
Now go to the SocketXP Portal's [Tunnel Page](https://portal.socketxp.com/#/tunnels).  Check the name of the tunnel created and its status.

### Run SocketXP Proxy Agent on your Laptop 
First download the SocketXP Proxy Agent in your laptop from the [SocketXP download page](https://www.socketxp.com/download).  SocketXP Proxy Agent is available for all OS versions - Windows/Mac/Linux.

Use the same SocketXP authtoken you used in the previous section to authenticate the proxy agent with SocketXP Cloud Gateway.

``` json
$ cat config.json
{
    "authtoken": "<your-auth-token-goes-here>"
    "tunnel_enabled": true, 
    "tunnels" : [{ 
        "destination": "tcp://postgres:5432", 
        "iot_device_id": "postgres-mservice-cluster1",
        "iot_slave": true
    }] 
}
```
``` sh
$ socketxp --config config.json
```

The above config requests socketxp agent to run in slave mode and create a local proxy TCP socket in your laptop at `127.0.0.1:5432`. Also it requests socketxp agent to create a secure TLS connection to the `postgres-mservice-cluster1` proxy device (running in your remote cluster or server) we created in the previous section.

>**Note:**
>Although it is called TCP socket, we run TLS on top of it.  So your data is securely transmitted over the internet end-to-end.

Next, update the DNS record in your laptop with the following command:
``` sh
$ echo "127.0.0.1    postgres" | sudo tee -a /etc/hosts > /dev/null
```
We need to add the above hostname mapping for our `postgres` microservice so that any local requests to the microservice will be routed to our SocketXP proxy agent listening at IP `127.0.0.1`.

### Run the python backend microserver
We are all set to run the python backend microservice under development in our laptop.

``` sh
$ python backend.py
Database opened successfully
Table created successfully
Student records added to the table successfully
```
Let's validate this success by running an SQL query in our remote postgres database.

``` sql
$ docker exec -it postgres psql -U gvelrajan postgres
postgres=# \c postgres
postgres=# \d
          List of relations
 Schema |  Name   | Type  |   Owner   
--------+---------+-------+-----------
 public | student | table | gannygans
(1 row)
postgres=# select * from student;
 admission | name | age |                       course                       |                     department                     
-----------+------+-----+----------------------------------------------------+----------------------------------------------------
     34231 | Dave |  19 | Information Technology                             | ICT                                               
     34232 | Gary |  18 | Electrical Engineering                             | ICT                                               
     34233 | Abel |  17 | Computer Science                                   | ICT                                               
(3 rows)

(END)
```
Awesome! We are able to develop/test our local python backend service and make it to seamlessly connect with the remote `postgres` DB microservice.

## Make a remote NGINX microservice locally accessible
Now suppose, in addition to the postgres db microservice, we also need to make an NGINX microserice locally accessible from our laptop, use the following `config.json` file to setup the SocketXP Proxy Agent in the remote server/cluster.

``` json
$ cat config.json
{
    "authtoken": "<your-auth-token-goes-here>"
    "tunnel_enabled": true, 
    "tunnels" : [
      { 
        "destination": "tcp://postgres:5432", 
        "iot_device_id": "postgres-mservice-cluster1", 
      },
      { 
        "destination": "tcp://nginx:80", 
        "iot_device_id": "nginx-mservice-cluster1", 
      },
    ] 
}
```
Next, go to your laptop, and update the config.json file to create a local Proxy for the nginx microservice, as shown below.

``` json
$ cat config.json
{
    "authtoken": "<your-auth-token-goes-here>"
    "tunnel_enabled": true, 
    "tunnels" : [
      { 
        "destination": "tcp://postgres:5432", 
        "iot_device_id": "postgres-mservice-cluster1",
        "iot_slave": true 
      },
      { 
        "destination": "tcp://nginx:80", 
        "iot_device_id": "nginx-mservice-cluster1", 
        "iot_slave": true 
      },
    ] 
}
```

``` sh
$ socketxp --config config.json
```
The above config will create a TCP listening socket at port 80 in your laptop.  Make sure port 80 is free in your laptop and not used by any other application.

Next, add the follow DNS entry into your `/etc/hosts` file, so that any local DNS resolution requests to access your `ngnix` service will be routed to the SocketXP Proxy Agent listening on IP 127.0.0.1. 

``` sh
$ echo "127.0.0.1    nginx" | sudo tee -a /etc/hosts > /dev/null
```

Now open up a browser in your laptop and point it to: http://nginx:80 or simply http://nginx.  It will open up the NNGINX home page as shown below.

![socketxp microservice remote access](https://dev-to-uploads.s3.amazonaws.com/i/naop7dmmxxkordq6uqdh.png)


## Share Same Set of Microservices Between Multiple Developers:
SocketXP Microservice Remote Access solution allows multiple developers to share same set of microservices running in a development server or K8s cluster.  You can create a many-to-one channel for each microservice that needs to be accessed remotely.

In SocketXP parlance, the SocketXP service proxy agent running as a standalone deployment along side a microservice (that needs to be accessed remotely) in a K8s cluster is called the master.  And the SocketXP service proxy agent running in the developer's laptop is called the slave.  Many slave instances of SocketXP proxy agent could run simultaneously on different developer's laptop and connect using a secure channel to the master instance of SocketXP proxy agent.

For the example usecase discussed in this document, each developer would run a SocketXP proxy agent on their laptop with the following config.json file to connect to the  Postgresql DB and Nginx microservice running in your remote server/cluster.  This communication channel goes via the single instance of the SocketXP proxy agent running in your remote server or cluster.  

``` json
$ cat config.json
{
    "authtoken": "<your-auth-token-goes-here>"
    "tunnel_enabled": true, 
    "tunnels" : [
      { 
        "destination": "tcp://postgres:5432", 
        "iot_device_id": "postgres-mservice-cluster1",
        "iot_slave": true 
      },
      { 
        "destination": "tcp://nginx:80", 
        "iot_device_id": "nginx-mservice-cluster1", 
        "iot_slave": true 
      },
    ] 
}
```

Basically a many-to-one secure communication channel is created to aid multiple developers share the same set of microservices running in a remote server or cluster.

## Kubernetes Microservice Remote Access:
So far, in our dicussion we have run the SocketXP proxy agent as a Docker container or as a binary natively.  We could potentially deploy SocketXP proxy agent as a Kubernetes Deployment to remote access microservices running in a K8s cluster.

To remotely access a microservice running in a Kubernetes(K8s) cluster or a minikube cluster, you need to run the SocketXP proxy agent as a Standalone Deployment in the cluster.

Instructions to run SocketXP proxy agent Docker container as a standalone deployment in a Kubernetes cluster can be found in this documentation page:

https://www.socketxp.com/docs/guide/kubernetes-cluster-remote-access.html#standalone-container

The only difference is that you need to use the following `config.json` file(same as the one used earlier above):  

``` json
$ cat config.json
{
    "authtoken": "<your-auth-token-goes-here>"
    "tunnel_enabled": true, 
    "tunnels" : [
      { 
        "destination": "tcp://postgres:5432", 
        "iot_device_id": "postgres-mservice-cluster1", 
      },
      { 
        "destination": "tcp://nginx:80", 
        "iot_device_id": "nginx-mservice-cluster1", 
      },
    ] 
}
```
The above config.json file assumes that your Postgresql DB and Nginx microservices are accessible within the cluster using the DNS names `postgres` and `nginx`.

::: warning Need Support?
We are looking for early users to evaluate our SocketXP Microservice Remote Access Solution.  Please write to us at: [support@socketxp.com](support@socketxp.com) if you have any questions or need any support.
:::