---
title: Microservices Architecture Pros and Cons
description: ""
author: Ganesh Velrajan
tags: [
    Arista Networks, Ethernet Switching, Programmable Ethernet Switch Chips, Cisco
]
date: 2018-07-22
categories: [
    Tech Industry, Architectures
]
images: ["/images/microservice/microservice.jpg"]
---

Microservices Architecture has become a buzz word in the industry creating so much hype around it.

Many organizations are just puzzled and in fact, feel guilty that they have been following a completely wrong architecture all along.

Kubernetes, LXC, Docker, Docker Swarm, and Apache Mesos are few of the technologies behind this Microservice Architecture hype wave.

Netflix, Amazon, Google, Microsoft, LinkedIn, and Facebook are some of companies driving this hype wave in the industry:  "It is the best architecture in the world.  It works for us! It must work for you too!"

It's high time we analyze the pros and cons of the Microservices Architecture to help those helpless souls left out wondering "Should we adapt Microservices Architecture or Not?"

In this article, we'll analyze and argue why Microservices Architecture is not ideal for many organizations and what are some of the Microservices architecture pros and cons

## What is a Microservices Architecture {#what-is-a-microservices-architecture .western}

Microservices Architecture is a modern way of building applications by breaking down a huge monolithic application into many small “micro” blocks such that each microblock focusses on providing just one functionality or “service” through well-defined interfaces such as REST, API, or a web service.

## Where was Microservices Architecture born {#where-was-microservices-architecture-born .western}

Microservices Architecture was born in hyperscale web2.0 companies such as Amazon, Google, Facebook, LinkedIn, eBay, Netflix and so on, which focusses on providing online services to almost the entire world.

## Why did hyperscalers choose Microservices Architecture {#why-did-hyperscalers-choose-microservices-architecture .western}

Hypersclaers chose Microservices Architecture over Monolithic Architecture for the following reasons:

-   Maintaining a single large monolithic application with so many tightly coupled software modules was very difficult.

-   Debugging problems in a monolithic application became a nightmare, when the sequence of function calls spanned more than one module of the application.

-   With more than 1000 engineers working on a single application, the hyperscalers found it very hard to prevent the affect of a software change done by one team in one module on the rest of the modules in the application.

-   Moreover, predicting the release date of an application software became almost impossible. Critical bugs found in just one module, close to a release date (release stopper or gating bugs), affect the release date of the entire application.

-   Couldn't scale up or scale down the monolithic application based on demand. A new instance of the entire application stack needs to be created even though the demand is high only for just one of the many functionalities provided by the application. Scaling up or scaling down a specific module that provides a specific functionality in an monolithic application was not possible.

## Microservices Architecture Pros and Cons

### What are the pros of using Microservices Architecture {#what-are-the-pros-of-using-microservices-architecture .western}

-   It is very easy to design, develop, maintain and debug a very small, completely independent application module that has just one service to provide to rest of the application using a well-defined interface.

-   With the boundaries and interfaces between microservices well-defined, organizations with a large development team, say in 1000's, can now divide the one big team into small teams that develop, test and own just their microservice module alone.

-   Each microservice module of an appliation can be released completely independent of the rest of the application modules.

-   Each microservice module can be upgraded in the production network completely independent of the rest of the application modules.

-   And more importantly, each microservice module can be scaled up or scaled down completely independent of the rest of the application modules.

### What are the cons of using Microservices Architecture {#what-are-the-cons-of-using-microservices-architecture .western}

-   The biggest disadvantage or overhead of Microservices Architecture is the requirement to define a well-defined interface for communication between different microservices of an application. Creating a well-defined interface between one microservice to another involves a overhead and costs some performance impact. For example, using a REST API to talk to a microservice is much slower than invoking a direct function call to a module or a service. This is because the microservices need to convert data back-and-forth between the REST format and the app native format, to communicate with each other.

-   Organizations with small development team cannot afford to have such a small resource grouped on a per microservice module basis. Moreover, such small development teams cannot afford to operate in independent silos.

-   It doesn't make sense to independently release or upgrade an application that doesn't have such a requirement. For instance, not all applications undergo frequent changes. Many organizations don't touch a stable application for years, if it works just fine.

## Why Microservices Architecture is not ideal for many organization {#why-microservices-architecture-is-not-ideal-for-many-organization .western}

The functional requirments of various applications developed and used by many organizations do not resemble those of the applications developed and used by hyperscalers. Microservices Architecture is not a panacea for all application problems. For many organizations building a monolithic application still makes sense. The cons of the Microservices architecture listed above may overshadow any pros of the architecture realized by such applications.

> Nevertheless, applications built on monolithic architecture, could still benefit from advances in server virtualization technologies such as containerization, and advances in devops best practices such as continuous integration and continuous delivery.

## 

## Conclusion: {#conclusion .western}

In conclusion, Microservice architecture shouldn't be blindly inherited into all applications developed in an organization. You need to analyze them on a case-by-case basis, weigh the pros and cons, and make a decision on the architecture. The pros and cons of the Microservices architecture listed in this article could guide your organization in that process.
