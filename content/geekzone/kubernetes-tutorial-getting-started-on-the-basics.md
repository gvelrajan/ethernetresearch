---
title: Kubernetes Tutorial - Getting Started on the Basics
description: ""
author: Ganesh Velrajan
tags: [
    Container, Docker, Kubernetes
]
date: 2018-07-22
categories: [
    GeekZone, Kubernetes
]
images: ["/images/kubernetes/kubernetes.jpg"]
---

The goal of this Kubernetes Tutorial is to get started on the basics.  We'll explore the various building blocks of Kubernetes and the terminologies used.

## What is Kubernetes: {#what-is-kubernetes .western}

Kubernetes is a container orchestration or management platform. Kubernetes is not an alternative to Docker container. Kubernetes actually used Docker to spawn containers in Kubernetes cluster nodes.

Kubernetes is an alternative to Docker Swarm or Apache Mesos.

### What is a Kubernetes Cluster: {#what-is-a-kubernetes-cluster .western}

![](http://35.238.255.76/wp-content/uploads/2018/07/module_01_cluster-300x243.jpg){.size-medium .wp-image-1031 .aligncenter width="300" height="243"}

A Kubernetes Cluster is a group of nodes that are connected together to run Kubernetes Pods in them. The nodes of a cluster, share the load of the pods running in them.

### What is a Kubernetes Node:

![](http://35.238.255.76/wp-content/uploads/2018/07/module_03_nodes-300x257.jpg){.size-medium .wp-image-1032 .aligncenter width="300" height="257"}

A Kubernetes Node is a physical or Virtual Machine, depending on the clusters.

There are two types of nodes – master nodes and worker nodes.  Master nodes run the Kubernetes controller and management software.  Worker nodes run Kubelet, the Kubernetes agent software.  Worker nodes also run multiple Kubernetes Pods in them.  Master nodes generally don't run pods in them, unless explicitly configured to do so.

### What is a Kubernetes Pod: {#what-is-a-kubernetes-pod .western}

![](http://35.238.255.76/wp-content/uploads/2018/07/module_03_pods-300x124.jpg){.size-medium .wp-image-1033 .aligncenter width="300" height="124"}

A Kubernetes Pod is a group of one or more application containers that share jstorage volumes, IP address, and information about how to run them.

### What is a Kubernetes Deployment {#what-is-a-kubernetes-deployment .western}

A Kubernetes Deployment is a group of replicas of a pod. Deployment has a deployment configuration file in YAML format. The Deployment configuration instructs Kubernetes how to create, run and update instances of your applications or pods.

For example, if an application requires 5 instances of an NGINX pod, the user creates a Deployment config file and specifies the NGINX app, the version of the app,  and the replicas count as 5. Kubernetes Deployment Controller will then spawn 5 instances of the pod in various worker nodes of the Kubernetes cluster.

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      labels:
        app: nginx
    spec:
      replicas: 5
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1.7.9
            ports:
            - containerPort: 80

The controller will monitor the Deployment and if a pod goes down, it will respawn a new one.

### What is a Kubernetes Service {#what-is-a-kubernetes-service .western}

When a Deployment has more than one replica of a pod, each pod will have its own local IP address and the port number on which the application runs.  In order for the Deployment to expose a single interface to other applications that depend on the service provided by the Deployment, it needs a service abstraction.  This service abstraction layer is called the Kubernetes Service.  The service abstraction layer can contain a port, IP address, and/or a load-balancer.

The Kubernetes Service is configured using a configuration file, again an YAML file. The service config file specifies the load-balancer and the IP address of the service.

This is the basic principle of microservices architecture.

A sample Service configuration file is shown below.

    kind: Service
    apiVersion: v1
    metadata:
      name: my-service
    spec:
      selector:
        app: MyApp
      ports:
      - protocol: TCP
        port: 80
        targetPort: 9376
      clusterIP: 10.0.171.239
      loadBalancerIP: 78.11.24.19
      type: LoadBalancer
    status:
      loadBalancer:
        ingress:
        - ip: 146.148.47.155

 

That's all folks about the basics on Kubernetes terminologies.

In the next tutorial, we'll show you [how to install Kubernetes on Ubuntu](http://35.238.255.76/index.php/2018/07/19/kubernetes-tutorial-how-to-install-kubernetes-on-ubuntu/).
