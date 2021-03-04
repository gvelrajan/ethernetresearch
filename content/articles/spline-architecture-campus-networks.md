---
title: Why Use Spline Architecture For Campus Networks
description: ""
author: Ganesh Velrajan
tags: [
    Arista Networks, campus switching, Cisco, Spline architecture
]
date: 2018-06-21
categories: [
    Tech Industry,
]
images: ""
---

Arista recently announced a new set of spline switches, Arista 7050X3 and 7300X3 for the campus switching market segment.  Arista also plans to disrupt the campus switching market segment by recommending Spline architecture for the campus switching network.  In this article, we'll investigate why Spline architecture, typically used in enterprise or cloud datacenter network, could potentially be used in enterprise campus network as well.

The datacenter switch chip technology has evolved significantly in the last decade such that a single switch chip can now support an entire campus network of a medium sized business.  Before we jump into the Spline architecture and how it is better than the classic 3-tier architecture, let's get the basics of Leaf-Spine architecture straight first.

#### What is a leaf-spine architecture:

-   Leaf-Spine architecture, a 2-tier architecture, is a way of interconnecting the switches in a network such that every leaf switch is connected to every spine switch in the network and vice versa.  However, there is no direct connectivity between spine switches.  Similarly, there is no direct connectivity between leaf switches in the network, as well.  A leaf switch talks to other leaf switches through spine switches only.
-   The beauty of this leaf-spine architecture is that each server connected to a leaf node is three hops away from other servers connected to other leaf nodes in the network.  Moreover, all the hosts can talk to every other hosts at line rate.
-   Leaf-Spine architecture, also called the CLOS architecture, is the one used to interconnect switch chips inside most modular switches being built for more than a decade now.  For instance, Cisco Catalyst 6500, Arista 7500 and Brocade XMR/MLX switches are built using CLOS architecture as the fundamental design construct.
-   To connect a large number of servers in a datacenter, we have two choices:
    -   buy a big modular switch that's built based on the CLOS architecture (switch chips connected in a leaf-spine fashion) **internally**.
    -   buy multiple small fixed form factor switches and form the CLOS architecture **externally**.

#### Why do you need a leaf-spine architecture:

-   Leaf-Spine architecture is ideal for a network with more east-west traffic than north-south traffic, which is the case with most datacenter networks.
-   In a typical cloud-scale datacenter network with 100,000+ servers, it is not possible to connect all the 10,000+ servers using a single big switch.  This is because no one builds such a big switch today.  It doesn't make business sense to build such a big switch.  Moreover, a customer would never know in advance how big of a switch they should buy, so that it would meet the future growth of the company.  In most cases, a customer may end up buying a big switch that ends up being under-utilized significantly.  Therefore, it is ideal for customers to buy multiple small switches and interconnect them in (Leaf-Spine) CLOS fashion **externally**, as opposed to buying one big switch that uses the same CLOS architecture to interconnect multiple switch chips **internally**.  As the business grows, the customer could buy more small switches and extend the leaf-spine network, to provide connectivity to more servers.

![arista-network-design](https://2eof2j3oc7is20vt9q3g7tlo5xe-wpengine.netdna-ssl.com/wp-content/uploads/2013/11/arista-network-design.jpg)

#### What is a spline architecture:

-   Spline architecture is a single-tier leaf-spine architecture in which each node does the function of both a leaf node and a spine node, hence the name "Spline".
-   Spline architecture eliminates the need to have an extra tier in the network design.
-   In spline architecture, every host could reach every other hosts in just one hop at line rate.  As a result, the packet latency is reduced.
-   Spline architecture is used when a single switch is sufficient to connect all the servers in a datacenter.  Another spline switch may be added to provide redundancy.  Most medium-sized enterprise DC needs can be met using a single Spline switch.

#### How Spline architecture eliminates the extra tier in Leaf-Spine architecture:

Typically, a cloud-scale datacenter would have a large number of servers(100,000+) to interconnect.  A single Spline switch cannot meet the needs of a hyperscale datacenter.  Hyperscale datacenters need a 2-tier leaf-spine architecture to make all those servers talk to each other.  As we discussed above, it is not possible to interconnect all the servers using a single big switch.  However, in a typical enterprise datacenter with fewer servers(1000+) it is possible to interconnect all the servers using a single modular switch.  This switch was not originally built for the "spline" use-case.  It was built to act like an End-Of-Rack(EOR) switch that interconnects all the Top-Of-Rack(TOR) switches - again in a Leaf-Spine fashion - at cloud-scale datacenter networks.  However, it's being repurposed for the "spline" use-case in the campus networks.

With higher radix(fancy name for large fan-outs) switch chips coming to the market, the spline switch need not be a modular switch but could potentially be a fixed form 2RU switch, with breakout cables used to connect each 100G ports to multiple 25G or 10G servers potentially.

#### Why Arista Networks is proposing the spline architecture for enterprise campus networks:

-   The answer is: "because they could".  The switch chip technology enables them to do so.

![](https://3s81si1s5ygj3mzby34dq6qf-wpengine.netdna-ssl.com/wp-content/uploads/2018/01/broadcom-trident-tomahawk-roadmap.jpg)

Fig 1: Growing bandwidth in a single chip.  Credits to "nextplatform.com"

-   From the figure-1 above, we could clearly see that a Tomahawk2 chip provides 6.4Tbps of bandwidth.  Today, a 2RU fixed form factor switch made of single Tomohawk2 chip could literally provide a fan-out of 64 x 100G ports or 640 x 10G ports(using breakout cables).  In comparison, in 2010, we would have required more than 10 Trident chips(more chips needed to form the spine layer in the 2-tier CLOS architecture) to provide the same fan-out.  More chips require more space, more power, and more cooling.  However, modern chips are getting better in all these aspects.
-   Arista and others could repurpose their datacenter switches for enterprise campus switching use-case.  For instance, a single modular Arista 7300X3 Spline switch(built based on CLOS architecture internally), providing upto 50Tbps of bandwidth, could potentially connect 5000+ WiFi access points and VOIP phones in a campus network.  When Tomohawk3 reaches the market, this number will be doubled.
-   Moreover, this large fan-out doesn't come at the compromise of line-rate traffic.  Arita 7300X3 is NOT a oversubscribed device, unlike Cisco Catalyst 9400 which is 3:1 oversubscribed.
-   Again, in the spline architecture, packets from end hosts have to go through lesser number of hops compared to the classic 3-tier campus architecture(access, aggregation, and core).

#### Recommendations to Enterprise customers:

Spline architecture is very promising given the chip technology that is driving this.  I foresee that in the future more campus switching networks will adapt to the spline architecture.  However, the only challenge I notice is that no single vendor has the capability to provide a complete campus switching solution today using the spline architecture, except Cisco.

I strongly believe that Cisco will not provide a campus switching solution based on the spline architecture anytime soon, given the investments they have already made in the Catalyst 9000 campus switching product line & DNA Center.  Moreover, Catalyst9Ks belong to the Enterprise Business Unit within Cisco and Nexus9Ks belong to the Datacenter Business Unit within Cisco.  Many a times, these business units within Cisco don't operate like a single company but treat each others as competitors. Therefore, the enterprise business unit guys wouldn't let the datacenter business unit guys to eat their lunch.

If you are an enthusiastic enterprise customer planning to deploy a spline architecture based campus switching network using Arista/Aruba/VMware based solution, I would recommend you to be very cautious about how the entire solution has been put together, as there are 2 different vendor devices and 3 different management softwares involved - of which 2 are proprietary closed solution.  The question you should ask these vendors is: who'll own the complete end-to-end solution and which single phone number should you call if there is any problem with the solution".
