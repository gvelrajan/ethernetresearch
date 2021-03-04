---
title: Kubernetes Tutorial - Hands On For Beginners
description: ""
author: Ganesh Velrajan
tags: [
    Container, Docker, Kubernetes
]
date: 2018-08-11
categories: [
    GeekZone, Kubernetes
]
images: ["/images/kubernetes/kubernetes.jpg"]
---

In the previous two Kubernetes tutorials, we saw how to install and setup a Kubernetes cluster and we discussed the fundamental building blocks of Kubernetes.

In this tutorial, we'll do a hands-on session on how to run a web application as a docker container in the  [Kubernetes Cluster we created](http://35.238.255.76/geekzone/kubernetes-tutorial-how-to-install-kubernetes-on-ubuntu/) previously.

## Sample Web Application

Here is the sample  "Hello, World" Node.js web application that we'll use for this exercise.

    $ cat myapp.js
    var http = require('http');
    var rand = Math.floor(Math.random() * 100);
    //create a server object:
    http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/html'});
    res.write('Hello World!'); //write a response to the client
    res.write('My number is: ' + rand); //write a response to the client
    res.end(); //end the response
    }).listen(80); //the server object listens on port 80

Here is the Dockerfile we'll use to package this JavaScript application into a docker container image.

    $ cat Dockerfile
    FROM alpine:latest
    RUN apk update && apk add nodejs
    RUN mkdir -p /usr/src/app
    COPY ./myapp.js /usr/src/app
    WORKDIR /usr/src/app
    EXPOSE 80 
    CMD ["node","myapp.js"]

Refer to the Docker Tutorial on [How to build a Docker Image](http://35.238.255.76/geekzone/docker-container-tutorial-creating-docker-container-images/), that we covered earlier.

## Build Docker Image

Build the docker image using the below command.  Replace the registry name "gvelrajan" with your own registry in DockerHub.

    $ docker image build -t gvelrajan/hello-world:v1.0 .

Once the image is built, push the image to the DockerHub, so that you can access it from anywhere.

    $ docker push gvelrajan/hello-world:v1.0

## Run the Docker Image in Kubernetes Cluster

Kubernetes is so popular because it makes the job of managing containers very easy.  All you need to do is describe your intent on "How to launch your web application" in a YAML file and Kubernetes will manifest that intent for you.  Kubernetes hides all the implementation details from the user.

For example, to run our "Hello, World" docker image we just built, we need to create a Kubernetes Pod YAML file and provide it as an input to Kubernetes.

Kubernetes will take care of scheduling the pod in one of the available nodes in the cluster.

## Create a Pod

Here is our Pod YAML file for our "Hello, World" webapp.

    $ cat pod.yaml
    apiVersion: v1
    kind: Pod                                            # 1
    metadata:
      name: helloworld                                   # 2
      labels:                                            
        app: helloworld
    spec:                                                # 3
      containers:
        - image: gvelrajan/hello-world:v1.0              # 4
          imagePullPolicy: Always
          name: helloworld                               # 5
          ports:
            - containerPort: 80                          # 6

**Description of various fields in the YAML file:**

\#1 kind:

\#2 name & labels

\#3 spec

\#4 image

\#5 name

\#6 containerPort

Request the Kubernetes master node to create a Kubernetes Pod by executing  the following command in the master node.

    master$ kubectl create -f pod.yaml
    pod/helloworld created

Check if the pod has been created.

    master$ kubectl get pod
    NAME                          READY     STATUS    RESTARTS   AGE
    helloworld                    1/1       Running   0          10s

To know more about the pod, use the "Kubectl describe pod" command as shown below.

    master$ kubectl describe pod helloworld
    Name:         helloworld
    Namespace:    default
    Node:         instance-2/10.160.0.3
    Start Time:   Sat, 11 Aug 2018 05:14:09 +0000
    Labels:       app=helloworld
    Annotations:  
    Status:       Running
    IP:           192.168.56.18
    Containers:
      helloworld:
        Container ID:   docker://91458eb52daaff0eec5df25fd9da004e721e9c56e3b06f1715b31821178ed155
        Image:          gvelrajan/hello-world:v2.0
        Image ID:       docker-pullable://gvelrajan/hello-world@sha256:6d15f54fce820eb2628a943734a60cbc042eda1e696d97b6
    3f159f88b0269f91
        Port:           80/TCP
        Host Port:      0/TCP
        State:          Running
          Started:      Sat, 11 Aug 2018 05:14:14 +0000
        Ready:          True
        Restart Count:  0
        Environment:    
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-n8l4x (ro)
    Conditions:
      Type              Status
      Initialized       True 
      Ready             True 
      ContainersReady   True 
      PodScheduled      True 
    Volumes:
      default-token-n8l4x:
        Type:        Secret (a volume populated by a Secret)
        SecretName:  default-token-n8l4x
        Optional:    false
    QoS Class:       BestEffort
    Node-Selectors:  
    Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                     node.kubernetes.io/unreachable:NoExecute for 300s
    Events:
      Type    Reason     Age   From                 Message
      ----    ------     ----  ----                 -------
      Normal  Scheduled  24s   default-scheduler    Successfully assigned default/helloworld to instance-2
      Normal  Pulling    23s   kubelet, instance-2  pulling image "gvelrajan/hello-world:v2.0"
      Normal  Pulled     19s   kubelet, instance-2  Successfully pulled image "gvelrajan/hello-world:v2.0"
      Normal  Created    19s   kubelet, instance-2  Created container
      Normal  Started    19s   kubelet, instance-2  Started container
    master$

Each pod gets its own local IP address in Kubernetes.  The pod above has been assigned an IP address of 192.168.56.18.

What if we need to scale up the webapp we created ?  Do we execute the above pod creation command again to create one more instance of the same pod?

    master$ kubectl create -f pod.yaml
    Error from server (AlreadyExists): error when creating "pod.yaml": pods "helloworld" already exists

This is not the right way to scale up pod or containers in Kubernetes.

This will serve as a nice segway to our next concept, which is Deployment.

## Create a Deployment

Kubernetes uses a Deployment YAML file to specify how many replicas of a pod we need and it will take care of spawning that many number of pods across different nodes in the cluster.

We don't have to worry about how and where the pods needs to be instantiated.  Kubernetes will take care of it on our behalf.

Kubernetes will even spawn a new pod if an existing pod dies.

Here is our Kubernetes Deployment file to spawn two pods for our webapp.

    master$ cat deployment.yaml 
    apiVersion: extensions/v1beta1
    kind: Deployment                                          # 1
    metadata:
      name: helloworld
    spec:
      replicas: 2                                             # 2
      minReadySeconds: 15
      strategy:
        type: RollingUpdate                                   # 3
        rollingUpdate: 
          maxUnavailable: 1                                   # 4
          maxSurge: 1                                         # 5
      template:                                               # 6
        metadata:
          labels:
            app: helloworld                                   # 7
        spec:
          containers:
            - image: gvelrajan/hello-world:v2.0 
              imagePullPolicy: Always                         # 8
              name: helloworld 
              ports:
                - containerPort: 80

Request Kubernetes Master to deploy our application.

    master$ kubectl create -f deployment.yaml 
    deployment.extensions/helloworld created

check the status of the deployment.

    master$ kubectl get deployment
    NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    helloworld   2         2         2            1           24s

To know more about the Deployment, execute the "kubectl describe deployment" command.

    master$ kubectl get pods
    NAME                          READY     STATUS    RESTARTS   AGE
    helloworld                    1/1       Running   0          3h
    helloworld-59d46f94c5-ddmvr   1/1       Running   0          2h
    helloworld-59d46f94c5-w8gcx   1/1       Running   0          2h

We have 3 pods of the "Hello world" application running now, including the one we created manually in the previous section.

Let's delete that pod now.

    master$ kubectl delete pod helloworld
    pod "helloworld" deleted

    master$ kubectl get pods
    NAME                          READY     STATUS    RESTARTS   AGE
    helloworld-59d46f94c5-ddmvr   1/1       Running   0          2h
    helloworld-59d46f94c5-w8gcx   1/1       Running   0          2h

Get detailed info on these pods.

    master$ kubectl describe pods
    Name:           helloworld-59d46f94c5-ddmvr
    Namespace:      default
    Node:           instance-2/10.160.0.3
    Start Time:     Sat, 11 Aug 2018 05:50:01 +0000
    Labels:         app=helloworld
                    pod-template-hash=1580295071
    Annotations:    
    Status:         Running
    IP:             192.168.56.23
    Controlled By:  ReplicaSet/helloworld-59d46f94c5
    Containers:
      helloworld:
        Container ID:   docker://7068f2006645d21824a29fc25122195f96f8b926fb6c251c3ec9b0454f1c835b
        Image:          gvelrajan/hello-world:v4.0
        Image ID:       docker-pullable://gvelrajan/hello-world@sha256:a499383b22475f7b2ac805003a1c432f6d4a4ca7196e66a4dabb94f10dd061c2
        Port:           80/TCP
        Host Port:      0/TCP
        State:          Running
          Started:      Sat, 11 Aug 2018 05:50:11 +0000
        Ready:          True
        Restart Count:  0
        Environment:    
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-n8l4x (ro)
    Conditions:
      Type              Status
      Initialized       True 
      Ready             True 
      ContainersReady   True 
      PodScheduled      True 
    Volumes:
      default-token-n8l4x:
        Type:        Secret (a volume populated by a Secret)
        SecretName:  default-token-n8l4x
        Optional:    false
    QoS Class:       BestEffort
    Node-Selectors:  
    Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                     node.kubernetes.io/unreachable:NoExecute for 300s
    Events:          


    Name:           helloworld-59d46f94c5-w8gcx
    Namespace:      default
    Node:           instance-2/10.160.0.3
    Start Time:     Sat, 11 Aug 2018 05:50:01 +0000
    Labels:         app=helloworld
                    pod-template-hash=1580295071
    Annotations:    
    Status:         Running
    IP:             192.168.56.24
    Controlled By:  ReplicaSet/helloworld-59d46f94c5
    Containers:
      helloworld:
        Container ID:   docker://a21882e7dd55d831fe965dfe7dfeb325165bcc13326b4c31e9c1b11dabbbd75e
        Image:          gvelrajan/hello-world:v4.0
        Image ID:       docker-pullable://gvelrajan/hello-world@sha256:a499383b22475f7b2ac805003a1c432f6d4a4ca7196e66a4
    dabb94f10dd061c2
        Port:           80/TCP
        Host Port:      0/TCP
        State:          Running
          Started:      Sat, 11 Aug 2018 05:50:08 +0000
        Ready:          True
        Restart Count:  0
        Environment:    
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-n8l4x (ro)
    Conditions:
      Type              Status
      Initialized       True 
      Ready             True 
      ContainersReady   True 
      PodScheduled      True 
    Volumes:
      default-token-n8l4x:
        Type:        Secret (a volume populated by a Secret)
        SecretName:  default-token-n8l4x
        Optional:    false
    QoS Class:       BestEffort
    Node-Selectors:  
    Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                     node.kubernetes.io/unreachable:NoExecute for 300s
    Events:

Kubernetes assigns each pod its own unique local IP address.

Should the client remember the IP address of each of these pods to communicate with them?  Moreover the IP address is a local IP address and not a public IP address.

So how do we make clients talk to each of these pods?

That's where the Kubernetes Service comes in handy.

A Kubernetes Services creates an abstraction on top of all the pods that provides the same functionality, so that the clients are not aware of the multiple instances of the pod.  Kubernetes Service exposes  a single interface for clients to communicate with the pods.

Kubernetes Service provides a load-balancers as well as a NAT service.  The load-balancer helps in load-balancing the traffic equally between the pods.  The NAT service provides translation between the public IP and Port and cluster local IP and port.

## Create a Service

Next create a load-balancer and NAT service.

    master$ cat service.yaml 
    apiVersion: v1
    kind: Service              # 1
    metadata:
      name: helloworld-lb
    spec:
      externalIPs:
      - 10.160.0.3
      type: LoadBalancer       # 2
      ports:
      - port: 80               # 3
        protocol: TCP          # 4
        targetPort: 80         # 5
      selector:                # 6
        app: helloworld        # 7

Request Kubernetes to create the service.

    master$ kubectl create -f service.yaml 
    service/helloworld-lb created

Check the status of the service.

    master$ kubectl get service
    NAME            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
    helloworld-lb   LoadBalancer   10.98.193.112   10.160.0.3    80:31870/TCP   9s
    kubernetes      ClusterIP      10.96.0.1               443/TCP        34d

To know the details use the describe command.

    master$ kubectl describe service helloworld-lb
    Name:                     helloworld-lb
    Namespace:                default
    Labels:                   
    Annotations:              
    Selector:                 app=helloworld
    Type:                     LoadBalancer
    IP:                       10.98.193.112
    External IPs:             10.160.0.3
    Port:                       80/TCP
    TargetPort:               80/TCP
    NodePort:                   31870/TCP
    Endpoints:                192.168.56.23:80,192.168.56.24:80
    Session Affinity:         None
    External Traffic Policy:  Cluster
    Events:

## Wrap it up

Check the browser:

    $ cat test.sh

    for var in {1..100} 
    do 
        curl http://10.160.0.3
        echo "\n"
    done

    $ sh test.sh 
    \Hello World!My number is:    23

    Hello World!My number is:    26

    Hello World!My number is:    26

    Hello World!My number is:    23

    Hello World!My number is:    23

    Hello World!My number is:    26

    Hello World!My number is:    26

    Hello World!My number is:    23

    Hello World!My number is:    23

    Hello World!My number is:    23

    Hello World!My number is:    23

    Hello World!My number is:    23

    Hello World!My number is:    26

    Hello World!My number is:    26

    Hello World!My number is:    26

    ...
    ...

In the above output, we see that the "Curl" output alternates between two different random numbers - each representing a pod in the cluster.

This proves that the Kubernetes Service is load-balancing our requests to the 2 different Pods running in the cluster.
