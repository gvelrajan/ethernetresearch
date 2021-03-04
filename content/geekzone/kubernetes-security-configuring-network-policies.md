---
title: Kubernetes Security - Configuring Network Policies
description: ""
author: Ganesh Velrajan
tags: [
    Container, Docker, Kubernetes, Security
]
date: 2018-11-12
categories: [
    GeekZone, Kubernetes
]
images: ["/images/kubernetes/kubernetes.jpg"]
---

Kubernetes Network Policies are similar to  ACL's and Route Policies that one could configure in networking devices such as Routers, Switches and Firewalls.  If you are CCNA Certified or familiar working with Cisco routers, then Network Policy is  a familiar topic to you.

But, even otherwise, it is a simple and an easy topic to learn, using the learning materials prescribed below.

Kubernetes Network policies are rules configured in the Kubernetes Cluster to instruct Kubernetes to allow or deny any communication between pods, for security reasons.  You can even control or restrict communication between a pod and an external entity such as a webserver or a web application or an end user.

There are ingress network policies and egress network policies, similar to ingress ACL's and egress ACL's in routers.  It tells the direction (of the traffic) in which the rule must be applied on a pod.

By default, when no network policies are configured, Kubernetes permits all traffic by default.  It doesn't restrict any communication between pods or with any external agents outside the cluster, by default.

You need to configure network policies to begin adding rules to whitelist pods or any external agents to communicate with.  Anything that is not whitelisted will be prevented from communicating with the pods running in the cluster.

**Please Note**:  Merely configuring network policies doesn't guarantee that network traffic will be restricted.  You need to install a network plugin (Calico, Flannel etc) to take these rules and put them to work in the cluster.

There are several level at which network policies can be configured - at the cluster level or namespace level or at the pod level.

I found the following github documentation  and the CNCF presentation by Ahmet very useful.  It covers almost everything you need to know about Kubernetes Network Policies.

### GitHub:

<https://github.com/ahmetb/kubernetes-network-policy-recipes>

### CNCF Presentation - YouTube Video

https://www.youtube.com/watch?v=3gGpMmYeEO8
