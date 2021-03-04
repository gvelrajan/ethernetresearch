---
title: Kubernetes Horizontal Pod Autoscaler and Metric Server
description: ""
author: Ganesh Velrajan
tags: [
    Container, Docker, Kubernetes
]
date: 2018-10-24
categories: [
    GeekZone, Kubernetes
]
images: ["/images/kubernetes/kubernetes.jpg"]
---

## What is Horizontal Pod Autoscaler(HPA)

The Horizontal Pod Autoscaler automatically scales up or scales down the number of pods in a deployment based on some metric such as CPU usage or some other custom metric.

Here is a **formal definition** from the Kubernetes Official Documentation:

"The Horizontal Pod Autoscaler automatically scales the number of pods in a replication controller, deployment or replica set based on observed CPU utilization (or, with [custom metrics](https://git.k8s.io/community/contributors/design-proposals/instrumentation/custom-metrics-api.md) support, on some other application-provided metrics). Note that Horizontal Pod Autoscaling does not apply to objects that can’t be scaled, for example, DaemonSets.

The Horizontal Pod Autoscaler is implemented as a Kubernetes API resource and a controller. The resource determines the behavior of the controller. The controller periodically adjusts the number of replicas in a replication controller or deployment to match the observed average CPU utilization to the target specified by user."

![Kubernetes Horizontal Pod Autoscaler and metric-server](http://www.thetechzone.in/wp-content/uploads/2018/09/Screen-Shot-2018-09-28-at-8.24.06-AM-300x252.jpg){.alignnone .wp-image-43 width="464" height="390"}

In this article, we'll configure HPA to use the CPU utilization as a metric to automatically scale up or scale down pods in a deployment using Minikube.

## Metric-Server:

Metric-Server collects the resource utilization such as CPU or Memory utilization across the cluster.  It provides a metrics API through which other agents, such as the *"kubectl top* "command,  HPA, and even the users,  could fetch the resource utilization information to make various decisions.

### Installing and Configuring Metric-Server and HPA in the cluster

In this demo, I'll use a  single node Minikube Cluster running in a VM in Google Cloud Platform to demonstrate the power of Horizontal Pod Autoscaler.

Following the instructions in this article to [install Minikube in a VM or Laptop](http://www.ethernetresearch.com/kubernetes/kubernetes-how-to-install-minikube-in-a-vm/)

If you are planning to implement HPA autoscaler using Kubernetes Cluster, then follow the procedure in this [documentation on Core Metric Pipeline](https://kubernetes.io/docs/tasks/debug-application-cluster/core-metrics-pipeline/) to install the metric-server.

#### Step \#1:

Enable the metric-server in the Minikube cluster.  You need to install the following command as a root user.

    gannygans@minikube:~$ sudo minikube addons enable metrics-server
    metrics-server was successfully enabled

#### Step\#2:

Create a deployment with 1 pod using the below deployment YAML file.  We'll use the *gvelrajan/hello-world:v2.0* application from my public docker hub registry.

The pod runs a simple nodejs helloworld web server application that prints "Hello World!" on the browser when connected to the web server.

    gannygans@minikube:~$ cat deployment.yaml 
    apiVersion: extensions/v1beta1
    kind: Deployment                                          # 1
    metadata:
      name: helloworld
    spec:
      replicas: 1                                             # 2
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
              resources:
                requests:
                  cpu: 200m

    gannygans@minikube:~$ kubectl get pods
    No resources found.
    gannygans@minikube:~$ kubectl get deployment
    No resources found.
    gannygans@minikube:~$ kubectl create -f deployment.yaml
    deployment.extensions/helloworld created

    gannygans@minikube:~$ kubectl get pods
    NAME READY STATUS RESTARTS AGE
    helloworld-5cd647f57c-z9s87 1/1 Running 0 2m

#### Step \#3:

Let's create a load balancer service named "*helloworld-lb*" using the below service YAML.

    gannygans@minikube:~$ cat service.yaml
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

    gannygans@minikube:~$ kubectl get service
    NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
    helloworld-lb LoadBalancer 10.110.45.50 10.160.0.3 80:32741/TCP 3h
    kubernetes ClusterIP 10.96.0.1 <none> 443/TCP 15d

### Configure the Autoscaler:

Now, let's configure an Autoscaler for the deployment we have created.

We want the autoscaler to automatically spin-up a new pod in the deployment, when the CPU utilization goes above 2%.

And similarly, if the CPU utilization goes below 2%, we want the Autoscaler to automatically bring down a pod.

We  want autoscaler to always run atleast 1 pod even when the CPU utilization is 0%.

At the maximum, it can scale the number of pods to 10.

It is a good security practice to set the max limit on the pods, when configuring autoscaler.

If we don't specify the maximum number, the autoscaler could be attacked by a DoS attacker to spin up unlimited number of pods and consume all the resources available in the cluster, making the other applications running in the same cluster to starve for resources.

    gannygans@minikube:~$ kubectl autoscale deployment helloworld --cpu-percent=2 --min=1 --max=10
    horizontalpodautoscaler.autoscaling/helloworld autoscaled
    gannygans@minikube:~$

Now check the status of the Horizontal Pod Scaler for our deployment.

    gannygans@minikube:~$ kubectl get hpa
    NAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGE
    helloworld Deployment/helloworld <unknown>/2% 1 10 0 18s
    gannygans@minikube:~$ kubectl get hpa
    NAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGE
    helloworld Deployment/helloworld <unknown>/2% 1 10 0 26s
    gannygans@minikube:~$ kubectl get hpa
    NAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGE
    helloworld Deployment/helloworld 0%/2% 1 10 1 1m

**Note**: It may take few minutes for the HPA to collect the metrics from the metric-server.  HPA  doesn't poll the metric-server too often.  It polls metric-server only every minute.

### Automatic Pod Scale Up and Scale Down Demo

Crank up the CPU usage by repeatedly polling the web-interface in a loop using  a shell script.

    $cat test.sh
    for var in {1..10000} 
    do 
          curl http://35.200.250.14/ 
          echo "\n"
    done
    $

    $sh test.sh
    Hello World!

    Hello World!

    Hello World!

    Hello World!

    Hello World!

    Hello World!

    Hello World!

    Hello World!

    ...

    ...

**Note**: It may take few minutes for the HPA to detect and begin taking any action.

    gannygans@minikube:~$ kubectl get hpa
    NAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGE
    helloworld Deployment/helloworld 2%/2% 1 10 1 4m


    gannygans@minikube:~$ kubectl get hpa
    NAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGE
    helloworld Deployment/helloworld 12%/2% 1 10 1 4m


    gannygans@minikube:~$ kubectl get pods
    NAME READY STATUS RESTARTS AGE
    helloworld-5cd647f57c-bstxl 1/1 Running 0 23s
    helloworld-5cd647f57c-djxn5 1/1 Running 0 23s
    helloworld-5cd647f57c-l7v94 1/1 Running 0 23s
    helloworld-5cd647f57c-z9s87 1/1 Running 0 7m
    gannygans@minikube:~$

    gannygans@minikube:~$ kubectl get hpa
    NAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGE
    helloworld Deployment/helloworld 2%/2% 1 10 4 7m

Now, let's kill the script to bring down the CPU utilization.

    ...

    ...

    Hello World!

    Hello World!

    Hello World!

    ^C
    $
    $

Check if the autoscaler detects the drop in CPU utilization and starts to bring down the pods one by one, slowly.

**Note**: It may take few minutes for the HPA to detect and begin taking any action.

    gannygans@minikube:~$ kubectl get hpa
    NAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGE
    helloworld Deployment/helloworld 9%/2% 1 10 4 5m
    gannygans@minikube:~$ kubectl get hpa
    NAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGE
    helloworld Deployment/helloworld 3%/2% 1 10 4 6m
    gannygans@minikube:~$ kubectl get hpa
    NAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGE
    helloworld Deployment/helloworld 3%/2% 1 10 4 6m
    gannygans@minikube:~$ kubectl get hpa
    NAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGE
    helloworld Deployment/helloworld 2%/2% 1 10 4 7m
    gannygans@minikube:~$ kubectl get hpa
    NAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGE
    helloworld Deployment/helloworld 2%/2% 1 10 4 7m
    gannygans@minikube:~$

    gannygans@minikube:~$ kubectl get hpa
    NAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGE
    helloworld Deployment/helloworld 0%/2% 1 10 4 9m
    gannygans@minikube:~$ kubectl get pods
    NAME READY STATUS RESTARTS AGE
    helloworld-5cd647f57c-bstxl 1/1 Terminating 0 5m
    helloworld-5cd647f57c-djxn5 1/1 Terminating 0 5m
    helloworld-5cd647f57c-l7v94 1/1 Terminating 0 5m
    helloworld-5cd647f57c-z9s87 1/1 Running 0 12m

    gannygans@minikube:~$ kubectl get pods
    NAME READY STATUS RESTARTS AGE
    helloworld-5cd647f57c-z9s87 1/1 Running 0 13m
    gannygans@minikube:~$

Now to delete the HPA autoscaler, using the delete command.

    gannygans@minikube:~$ kubectl delete hpa helloworld
    horizontalpodautoscaler.autoscaling "helloworld" deleted

 

 

 

 

 

 
