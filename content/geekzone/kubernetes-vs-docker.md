---
title: Kubernetes vs Docker Swarm - 5 Major Differences
description: ""
author: Ganesh Velrajan
tags: [
    Container, Docker, Docker Swarm, Kubernetes
]
date: 2018-07-08
categories: [
    GeekZone, Kubernetes, Docker
]
images: ["/images/kubernetes/kubernetes.jpg"]
---

Here are the 6 differences between Docker Swarm and Kubernetes.  But before that let's refresh some basics on Docker and Kubernetes.  If you are in a haste, you can jump straight to **Kubernetes vs Docker Swarm** debate down below.

## What is Docker?

Docker  is a containerization platform.  It helps create Docker images and Docker containers from an application binary and configuration files.

Docker leverages the containerization primitives such as namespace, cgroups provided by the Linux Kernel to run docker containers.

## What is Docker Swarm

Docker Swarm is a proprietary management software from Docker to manage a swarm of containers running in a Docker cluster.

Docker Swarm is used to create and manage the cluster nodes as well schedule container jobs to run in the cluster nodes.

## What is Kubernetes or K8s?

Kubernetes (also known as K8s) is an open source "orchestration" or "management" platform for managing containers.

> Kubernetes is "NOT" a containerization platform that competes with Docker container engine(Docker CE/EE) .

It actually leverages Docker to instantiate containers in a cluster.  For example, Kubernetes requires you to install Docker CE/EE to run and manage your applications that are packaged as Docker images.

### What Kubernetes does?

Kubernetes creates and manages cluster nodes.  Kubernetes creates and manages pods containing containerised applications within those nodes in a cluster.

> In very simple terms, Kubernetes is more like OpenStack equivalent in the container space.

Kubernetes competes with Docker Swarm and Apache Mesos in the container cluster orchestration market.

## Kubernetes vs Docker Swarm - Differences

Here are a few primary differences between Kubernetes and Docker Swarm:

Kubernetes | Docker Swarm 
-----------|--------------
It is a free open source technology, originally developed at Google  | It is a proprietary solution developed and sold by Docker Inc.
It is a highly scalable platform. Kubernetes can manage upto 5000 nodes in a cluster and 150,000 pods within those cluster nodes. | Docker Swarm is relatively less scalable. Docker Swarm can support only 1000 nodes in a cluster and 30,000 pods with those cluster nodes.                                                    
Kubernetes cluster consists of Master nodes and Worker nodes | Docker Swarm consists of Manager nodes and Worker nodes
Supports High Availability(HA). Multiple Master nodes and Worker nodes can be configured. | Supports High Availability(HA). Multiple Manager nodes and Worker nodes can be configured.
Kubernetes terminologies: Pods, Deployments, Services | Docker Swarm terminologies: Stack, Services, Tasks 
In Kubernetes terminologies, a deployment is a collection or multiple replicas of the same pod. | In Docker Swarm terminologies, a Stack is a group of services that are usually deployed together, such as a frontend service and a backend service that together forms an application stack.

### Which one is popular Kubernetes or Docker Swarm?

Kubernetes is more powerful, robust and has wider industry adoption than Docker Swarm.  Moreover, it is a free open source software.

Kubernetes is supported by all the three major cloud service provides - AWS, MS Azure and Google Cloud Platform.

Kubernetes is providing a though competition to Docker.  The flurry of organizational changes happening within Docker company is due to these reasons.

 
