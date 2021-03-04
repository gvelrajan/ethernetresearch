---
title: Docker Container Tutorial - Docker Swarms
description: ""
author: Ganesh Velrajan
tags: [
    GeekZone, Docker, Kubernetes, Docker Swarm
]
date: 2018-07-19
categories: [
    GeekZone, Docker
]
images: ["/images/docker/docker.png"]
---

Docker Swarm is a very useful management tool when we have more than one container and more than one host to manage.

In other words, when we have a "swarm of containers" it is nearly impossible to manage each one of them individually.  We need a management software or tool to manage a swarm of containers.  Docker Swarm is a software tool that does the job.

In this tutorial, we'll discuss how to work with Docker Swarms to manage Docker Containers.

### Initializing Docker Swarm:

A Docker swarm consists of manager and worker nodes.  There can be multiple manager nodes to provide redundancy.  One of the manager nodes will become a leader node and all others will be in backup mode.  Manager nodes are used for configuring and managing the swarm nodes.   Similarly, there can be multiple worker nodes to load-balance the container workloads between them.

Let's initialize a Docker Swarm and create a manager node.  In this tutorial, we will have only one manager node and one worker node.  Each node can be a physical host or a virtual machine host.

    manager$ docker swarm init --advertise-addr $(hostname -i)
    Swarm initialized: current node (w69ubdlip5mou1dluuvcb76qx) is now a manager.

    To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-3kdognqu7fy07fxkp5jhl30752dhmxtdevg3bu3cfm64c185fm-8vtuu2y46dgtgxb8apajzeh0z 192.168.0.13:2377

    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

Next join the Docker swarm from the worker node using the token provided by the manager node.

    worker$ docker swarm join --token SWMTKN-1-3kdognqu7fy07fxkp5jhl30752dhmxtdevg3bu3cfm64c185fm-8vtuu2y46dgtgxb8apajzeh0z 192.168.0.13:2377
    This node joined a swarm as a worker.

List the nodes in the swarm by running the following command in the master node only.

    manager$ docker node ls
    ID HOSTNAME STATUS AVAILABILITY MANAGER STATUS ENGINE VERSION
    w69ubdlip5mou1dluuvcb76qx * node1 Ready Active Leader 18.03.1-ce
    2ojpw5poh6s33ep9e4o6q509v node2 Ready Active 18.03.1-ce

The above output shows that the manager node is also the leader node because there is only one manager node configured in the swarm.  Typically production environment would have multiple manager nodes for redundancy.

If we execute the same command in the worker node it will complain that it is not a master to execute any swarm management commands.

    worker$ docker node ls
    Error response from daemon: This node is not a swarm manager. Worker nodes can't be used to view or modify cluster state. Please run this command on a manager node or promote the current node to a manager.

 

### Clone the Voting App:

Let's download the sample voting app from the Github.  Please make sure you run this command in the master node.

    manager$ git clone https://github.com/docker/example-voting-app
    Cloning into 'example-voting-app'...
    remote: Counting objects: 482, done.
    remote: Total 482 (delta 0), reused 0 (delta 0), pack-reused 482
    Receiving objects: 100% (482/482), 229.43 KiB | 13.50 MiB/s, done.
    Resolving deltas: 100% (179/179), done.

    manager$ cd example-voting-app

    manager$ ls
    LICENSE docker-stack.yml
    MAINTAINERS dockercloud.yml
    README.md k8s-specifications
    architecture.png result
    docker-compose-javaworker.yml vote
    docker-compose-simple.yml worker
    docker-compose.yml

Let's check the contents of the docker-stack.yml file.  It will have instructions on how each services in a stack should be instantiated and configured.

    manager$ cat docker-stack.yml
    version: "3"
    services:

    redis:
     image: redis:alpine
     ports:
     - "6379"
     networks:
     - frontend
     deploy:
     replicas: 1
     update_config:
     parallelism: 2
     delay: 10s
     restart_policy:
     condition: on-failure
     db:
     image: postgres:9.4
     volumes:
     - db-data:/var/lib/postgresql/data
     networks:
     - backend
     deploy:
     placement:
     constraints: [node.role == manager]
     vote:
     image: dockersamples/examplevotingapp_vote:before
     ports:
     - 5000:80
     networks:
     - frontend
     depends_on:
     - redis
     deploy:
     replicas: 2
     update_config:
     parallelism: 2
     restart_policy:
     condition: on-failure
     result:
     image: dockersamples/examplevotingapp_result:before
     ports:
     - 5001:80
     networks:
     - backend
     depends_on:
     - db
     deploy:
     replicas: 1
     update_config:
     parallelism: 2
     delay: 10s
     restart_policy:
     condition: on-failure

    worker:
     image: dockersamples/examplevotingapp_worker
     networks:
     - frontend
     - backend
     deploy:
     mode: replicated
     replicas: 1
     labels: [APP=VOTING]
     restart_policy:
     condition: on-failure
     delay: 10s
     max_attempts: 3
     window: 120s
     placement:
     constraints: [node.role == manager]

    visualizer:
     image: dockersamples/visualizer:stable
     ports:
     - "8080:8080"
     stop_grace_period: 1m30s
     volumes:
     - "/var/run/docker.sock:/var/run/docker.sock"
     deploy:
     placement:
     constraints: [node.role == manager]

    networks:
     frontend:
     backend:

    volumes:
     db-data:

### Deploy a Stack:

A stack is a group of services that are usually deployed together, such as a frontend service and a backend service that together forms an application stack.  Each service can be made up of one or more container instances called tasks.  For example, there can be multiple instances of the frontend container spawned to handle more traffic. Each such container instance is a task of the frontend service.

Let's deploy the stack from the manager console using the docker-stack.yml file.

    manager$ docker stack deploy --compose-file=docker-stack.yml voting_stack
    Creating network voting_stack_backend
    Creating network voting_stack_default
    Creating network voting_stack_frontend
    Creating service voting_stack_vote
    Creating service voting_stack_result
    Creating service voting_stack_worker
    Creating service voting_stack_visualizer
    Creating service voting_stack_redis
    Creating service voting_stack_db

    manager$ docker stack ls
    NAME SERVICES
    voting_stack 6

Here is the command to list the services spawned for a given stack.  Note in the below output that the "voting_stack_vote" service has 2 replicas, as we have instructed in the docker-stack.yml file.  These two replicas are the 2 tasks of the "voting_stack_vote" service.

    manager$ docker stack services voting_stack
    ID NAME MODE REPLICAS IMAGE PORTS
    070sq2hxkv5i voting_stack_db replicated 1/1 postgres:9.4
    7p20869dlwdq voting_stack_vote replicated 2/2 dockersamples/examplevotingapp_vote:before *:5000->80/tcp
    lo0sdggwktfh voting_stack_redis replicated 1/1 redis:alpine *:30000->6379/tcp
    m02xt0qcqyvd voting_stack_visualizer replicated 1/1 dockersamples/visualizer:stable *:8080->8080/tcp
    maxc947eok0l voting_stack_result replicated 1/1 dockersamples/examplevotingapp_result:before *:5001->80/tcp
    smhw70zh0qb1 voting_stack_worker replicated 1/1 dockersamples/examplevotingapp_worker:latest

To display the replicas ( or tasks) associated with a service, execute the following command.

    manager$ docker service ps voting_stack_vote
    ID NAME IMAGE NODE DESIRED STATE CURRENT STATE ERROR PORTS
    aabui7rdmc4v voting_stack_vote.1 dockersamples/examplevotingapp_vote:before node1 Running Running 7 minutes ago
    jd1wzdyq3bw5 voting_stack_vote.2 dockersamples/examplevotingapp_vote:before node2 Running Running 7 minutes ago

If for some reason, after casting votes, people throng the result website "voting_stack_result" in the voting app above, we may have to scale up that service.  To scale up a service, increase the number of replicas or tasks or containers spawned for the service, using the following command.

    manager$ docker service scale voting_stack_result=5
    voting_stack_result scaled to 5
    overall progress: 5 out of 5 tasks
    1/5: running
    2/5: running
    3/5: running
    4/5: running
    5/5: running
    verify: Service converged

Similarly, you can scaled down a service by reduce the count in the above command.

    manager$ docker service scale voting_stack_result=3
    voting_stack_result scaled to 3
    overall progress: 3 out of 3 tasks
    1/3: running
    2/3: running
    3/3: running
    verify: Service converged

    manager$ docker stack services voting_stack
    ID NAME MODE REPLICAS IMAGE PORTS
    070sq2hxkv5i voting_stack_db replicated 1/1 postgres:9.4
    7p20869dlwdq voting_stack_vote replicated 2/2 dockersamples/examplevotingapp_vote:before *:5000->80/tcp
    lo0sdggwktfh voting_stack_redis replicated 1/1 redis:alpine *:30000->6379/tcp
    m02xt0qcqyvd voting_stack_visualizer replicated 1/1 dockersamples/visualizer:stable *:8080->8080/tcp
    maxc947eok0l voting_stack_result replicated 3/3 dockersamples/examplevotingapp_result:before *:5001->80/tcp
    smhw70zh0qb1 voting_stack_worker replicated 1/1 dockersamples/examplevotingapp_worker:latest

    manager$ docker ps
    CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
    791115ff9838 dockersamples/examplevotingapp_worker:latest "/bin/sh -c 'dotnet …" About an hour ago Up About an hour voting_stack_worker.1.ijtteerugzkrzf99mjcpqnwqm
    2e5d745cc699 postgres:9.4 "docker-entrypoint.s…" About an hour ago Up About an hour 5432/tcp voting_stack_db.1.yhby06qsc53ryks4at9xkhzsu
    9cb6543aa5ae dockersamples/visualizer:stable "npm start" About an hour ago Up About an hour 8080/tcp voting_stack_visualizer.1.z7937i0rkwi81p35zsn67s6fo
    7a57ad0a16a1 dockersamples/examplevotingapp_result:before "node server.js" About an hour ago Up About an hour 80/tcp voting_stack_result.1.ibhhdgrgqjfmsi4fgk4d2tnpe
    4c729d9cf487 dockersamples/examplevotingapp_vote:before "gunicorn app:app -b…" About an hour ago Up About an hour 80/tcp voting_stack_vote.1.aabui7rdmc4viz9t76x6k93dn

    worker$ docker ps
    CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
    079a5a4506dd dockersamples/examplevotingapp_result:before "node server.js" 9 minutes ago Up 9 minutes 80/tcp voting_stack_result.3.ddozkt7a528ek69479jl5woud
    8f25b2d0512f dockersamples/examplevotingapp_result:before "node server.js" 9 minutes ago Up 9 minutes 80/tcp voting_stack_result.2.5tmjvecjtlx8pnzy52y98dxx5
    5811c9b7ac84 redis:alpine "docker-entrypoint.s…" About an hour ago Up About an hour 6379/tcp voting_stack_redis.1.s6ky4lq93m3moa2oc7i5m612a
    7aa7718315d9 dockersamples/examplevotingapp_vote:before "gunicorn app:app -b…" About an hour ago Up About an hour 80/tcp voting_stack_vote.2.jd1wzdyq3bw5x26aq3jqlrvrb

### 

### Drain a node and pull it out of service:

In the real world, we need to shutdown nodes for upgrade and periodic maintenance.  Docker allows users to pull out nodes from a swarm and shut the node down.  It involves two steps:  first, we need to move the services away from the node to other nodes in the swarm; second, we need to pull out the node from the swarm.

To drain the services out of a node execute the following command.

    manager$ docker node update --availability drain 2ojpw5poh6s33ep9e4o6q509v
    2ojpw5poh6s33ep9e4o6q509v

check the worker node and see if the services have been drained out of it.

    worker$ docker ps
    CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES

Check the manager node to see if all the services from the worker node have been moved to it.

    manager$ docker ps
    CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
    1c8a7935a8dc dockersamples/examplevotingapp_worker:latest "/bin/sh -c 'dotnet …" 2 minutes ago Up 2 minutes voting_stack_worker.1.v2a312sb73eur5ncf5b4g6007
    3882b7a45c1c redis:alpine "docker-entrypoint.s…" 2 minutes ago Up 2 minutes 6379/tcp voting_stack_redis.1.gvys2udrcgcnm2eqfnbzj0jhg
    83f2126a17b6 dockersamples/examplevotingapp_vote:before "gunicorn app:app -b…" 2 minutes ago Up 2 minutes 80/tcp voting_stack_vote.2.qfwsop38s23iqfwiz762vbd91
    b17ba659bb3e dockersamples/examplevotingapp_result:before "node server.js" 2 minutes ago Up 2 minutes 80/tcp voting_stack_result.3.ot6qinxotkwxt9522ywg7swds
    75d2c009233f dockersamples/examplevotingapp_result:before "node server.js" 2 minutes ago Up 2 minutes 80/tcp voting_stack_result.2.pnzjtjjcrjvuvad5dg4dsfdvf
    2e5d745cc699 postgres:9.4 "docker-entrypoint.s…" About an hour ago Up About an hour 5432/tcp voting_stack_db.1.yhby06qsc53ryks4at9xkhzsu
    9cb6543aa5ae dockersamples/visualizer:stable "npm start" About an hour ago Up About an hour 8080/tcp voting_stack_visualizer.1.z7937i0rkwi81p35zsn67s6fo
    7a57ad0a16a1 dockersamples/examplevotingapp_result:before "node server.js" About an hour ago Up About an hour 80/tcp voting_stack_result.1.ibhhdgrgqjfmsi4fgk4d2tnpe
    4c729d9cf487 dockersamples/examplevotingapp_vote:before "gunicorn app:app -b…" About an hour ago Up About an hour 80/tcp voting_stack_vote.1.aabui7rdmc4viz9t76x6k93dn

Next we could pull out the worker node from the swarm.

    worker$ docker swarm leave
    Node left the swarm.

We can shutdown the worker node now for maintenance.

That's all folks !  You have become a Docker pro.  Now you know how to create and manage Docker Swarm.  Congratulations !!
