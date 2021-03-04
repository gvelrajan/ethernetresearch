---
title: A Market Study On Programmable Ethernet Switch Chips
description: ""
author: Ganesh Velrajan
tags: [
    Arista Networks, Ethernet Switching, Programmable Ethernet Switch Chips, Cisco
]
date: 2018-06-19
categories: [
    Tech Industry,
]
images: ""
---

Arista Networks recently introduced a new family of programmable switches named Arista 7170 Series, made of Barefoot Tofino based programmable Ethernet switch chips.  Arista, in December 2016, had introduced a similar family of programmable switches named Arista 7160 Series, made of XPliant XP80 programmable Ethernet switch chips from Cavium.

### Why is Arista earnestly and diligently building new family of switches made of switch chips from Cavium Xpliant, Barefoot and other fledgling switch chip manufacturers ? 

1.  Currently, Broadcom holds 96 percent market share in 10 gigabit and faster Ethernet chips, according to the market research firm The Linley Group.

2.  Arista is almost totally dependent on Broadcom's switch chips to build its datacenter switches because Arista doesn't make switch chips of its own, unlike Cisco and Juniper. It's a huge business risk for Arista to depend on just one vendor for all its switch chip supplies. So probably Arista wants to diversify its risks by procuring switch chips from these new switch chip startups as well.

3.  Moreover, Arista wants to improve its negotiating power with Broadcom.  Arista may be paying a premium to  procure the latest generation of switch chips from Broadcom because Arista does not have a choice.  However, Cisco,  Arista's prime competitor, partly makes its switch chips in house and partly buys it from Broadcom.  So it is in Arista's best interest to promote a healthy competition to exist in the switch chip manufacturing industry.

4.  In addition to the risk mitigation and negotiating power, Arista has something more to gain from using these programmable switch chips in its products, which is the programmability itself. Today, Arista, Cisco, Brocade/Extreme, Juniper and plenty of whitebox switch manufacturers all use the same Broadcom chips. There is no way for these players to create product differentiation in the hardware. With the advent of these programmable chips, Arista, and others, could potentially create a product that differentiates itself from other competitors. Note that only differentiated products could demand price premium.

5.  Arista is a white box manufacturer.  If they don't do it first, other whitebox manufacturers such as Delta, Quanta would eventually do it and become a threat to Arista.

### Why Broadcom still continues to make next-gen switch chips such as Tomohawk3 without a fully programmable pipeline, despite many competitors such as Xpliant/Cavium, Barefoot Networks, and Innovium emerging in this space ? 

### Why is Broadcom focussing primarily on the feeds and speeds ?

1.  Arista announced its first fully programmable switching platform Arista 7160 Series made of the Xpliant/Cavium switch chip in 2016. Since then how many of these boxes did Arista sell to its customers, primarily to the hyperscale datacenter operators such as Google, Facebook and Microsoft ? The answer is not that many.

2.  According to Rochan Sankar, senior director of core switch group at Broadcom, hyperscale cloud datacenter operators look for the feeds and the speeds and don't look for the programmability. Moreover, Sankar claims that they have studied the microarchitecture of programmable chips and it is not possible to build a fully programmable pipeline without compromising on the feeds and speeds.  The primary features hyperscalers look in a switch chip are the feeds and speeds, low power consumption and port density at the right economics. Effectively speaking, hyperscalers are not willing to pay a premium for the “programmability” aspect of the chip. If hyperscale cloud datacenter operators, the ones with the sophistication and the talent pool, doesn't want to leverage the programmability aspect of these new programmable switch chips then who else could  ?

3.  The only other potential market segment where programmable chips could be leveraged, in volumes comparable to the cloud datacenter market segment, is the enterprise data centre and campus switching market. This is because, this the market where switch vendors could heavily differentiate themselves and charge a price premium. Moreover, these programmable chips have the ability to mimic the functionality of a load-balancer or firewall in addition to the core packet forwarding functionality, thereby eliminating the need to have a separate load-balancer or firewall devices in the enterprise network.  Moreover, the advanced telemetry features helps provide visibility into the network. Cisco and HPE are the market leaders in the campus switching market segment. They use their in-house built switch chip in their campus products. Today, Broadcom is not a major player in the campus switching space. Effectively, Broadcom will be least affected by the programmable switch chip manufacturers in this market segment as well.

4.  It is quite possible that Broadcom is working on a fully programmable chip but doesn't want to talk about it now because it might affect the sales of its other exists chips such as Tomohawk or Trident etc.   Also the market is not yet ripe for the fully programmable switch chips, so Broadcom could delay bringing its version of the fully programmable switch chips to the market.

5.  Broadcom is always known to use price and promotion as a weapon to safeguard its market share from these fledgling switch chip startups.  Broadcom significantly reduces the price on its existing switch chips in the market or provide a huge discount for the upcoming ones, like the Tomohawk3, when a new and equally powerful competing switch chip comes to the market, like the Innovium TERALYNX switch chip.

### Why did Google, along with Goldman Sachs, lead the recent \$57 million investment in Barefoot networks ? If hyperscalers don't love the programmability aspect of the switch chips over the feeds and speeds, why is Google investing in Barefoot Networks ? 

1.  Nick McKeown,  the Chief Scientist and co-founder of Barefoot Networks, is a Stanford University professor.  Moreover, Barefoot CEO Craig Barratt is a Google Fiber alum.

2.  Google, like Arista, probably prefers to have a second vendor for the networking switch chips – to reduce cost and dependency on Broadcom.  Remember that Google builds its own whitebox switches for its own use.

3.  Google probably wants to experiment with these new programmable switch chips in its datacenter networks.  Google, Microsoft and other are keen to innovate new proprietary protocols that works for them, much faster than waiting for the entire networking community to standardize on an IETF draft.

4.  However, in the short-run, to meet its economics, Google needs switches that provide the feeds and speeds  and low power consumption, primarily.

5.  Tencent and Alibaba, both investors from China, were among the other investors in the recent round of investments in Barefoot networks.  May be Google was brought in to make sure that no single voice could heavily influence the direction of the company.  Most recently, Google and Tencent have been doing several joint investments in the Asian market.  Probably, they are following a similar approach in the U.S. market too.

Programmable switch chips are very promising.  However, their survival depends on the favourable support they receive from the industry players such as Amazon, Arista, Google, Microsoft and others.

Arista in introducing a new switch series, Arista 7170, has embraced the programmable switch chips.  Microsoft also seems have embraced the [programmable switch platforms](https://github.com/Azure/SONiC/wiki/Supported-Devices-and-Platforms) by supporting SONIC on them.  It is just a matter time before the fully-programmable switch chips could take over the market share from non fully-programmable Broadcom switch chips in the market.
