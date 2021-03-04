---
title: Linux Container (LXC) Networking Fundamentals
description: ""
author: Ganesh Velrajan
tags: [
    Linux Containers, LXC
]
date: 2016-11-18
categories: [
    GeekZone, Linux Containers, Networking
]
images: ["/images/linux/linux-container-lxc.jpeg"]
---

We saw how to create an LXC container in the [previous article](https://ganeshtechblog.wordpress.com/2016/11/05/beginners-guide-to-linux-containers-with-lxc/).  In this article we'll see how to set up a basic network for LXC and make it communicate to the outside world.

## LXC Basic Networking Setup:

An LXC Container comes up with default networking interfaces - an ethernet interface(eth0)and a loopback interface(lo).  I logged into my "*test-container"* created in the [previous article ](https://ganeshtechblog.wordpress.com/2016/11/05/beginners-guide-to-linux-containers-with-lxc/)and listed the interface using the *ifconfig* command.

    root@test-container:/# ifconfig
    eth0      Link encap:Ethernet  HWaddr 00:16:3e:b4:86:23  
              inet addr:10.0.3.124  Bcast:10.0.3.255  Mask:255.255.255.0
              inet6 addr: fe80::216:3eff:feb4:8623/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:25 errors:0 dropped:0 overruns:0 frame:0
              TX packets:11 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000 
              RX bytes:4801 (4.8 KB)  TX bytes:1382 (1.3 KB)

    lo        Link encap:Local Loopback  
              inet addr:127.0.0.1  Mask:255.0.0.0
              inet6 addr: ::1/128 Scope:Host
              UP LOOPBACK RUNNING  MTU:65536  Metric:1
              RX packets:0 errors:0 dropped:0 overruns:0 frame:0
              TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:0 
              RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)


    root@test-container:/#

The *eth0* interface has been assigned an private IP address of 10.0.3.124. This LXC subnet doesn't overlap with the host network subnet address. The *ifconfig* command executed on the host machine ( outside the container ) provides the following output.

    $ ifconfig
    eth0      Link encap:Ethernet  HWaddr 00:0c:29:9c:80:88  
              inet addr:192.168.85.132  Bcast:192.168.85.255  Mask:255.255.255.0
              inet6 addr: fe80::20c:29ff:fe9c:8088/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:252 errors:0 dropped:0 overruns:0 frame:0
              TX packets:298 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000 
              RX bytes:67077 (67.0 KB)  TX bytes:31026 (31.0 KB)

    lo        Link encap:Local Loopback  
              inet addr:127.0.0.1  Mask:255.0.0.0
              inet6 addr: ::1/128 Scope:Host
              UP LOOPBACK RUNNING  MTU:65536  Metric:1
              RX packets:276 errors:0 dropped:0 overruns:0 frame:0
              TX packets:276 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:0 
              RX bytes:39984 (39.9 KB)  TX bytes:39984 (39.9 KB)

    lxcbr0    Link encap:Ethernet  HWaddr fe:d9:bb:5b:a6:81  
              inet addr:10.0.3.1  Bcast:10.0.3.255  Mask:255.255.255.0
              inet6 addr: fe80::744d:3ff:fef2:5c6e/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:147 errors:0 dropped:0 overruns:0 frame:0
              TX packets:183 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:0 
              RX bytes:12744 (12.7 KB)  TX bytes:26509 (26.5 KB)

    veth3M3GH3 Link encap:Ethernet  HWaddr fe:d9:bb:5b:a6:81  
              inet6 addr: fe80::fcd9:bbff:fe5b:a681/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:48 errors:0 dropped:0 overruns:0 frame:0
              TX packets:71 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000 
              RX bytes:4592 (4.5 KB)  TX bytes:10258 (10.2 KB)

    $ 

In the above output, we see *eth0* and *lo* as usual. However, the IP address configured on the physical ethernet interface *eth0* is 192.168.85.132.  Interestingly, the lxcbr0 has been assigned an IP address of 10.0.3.1 which falls in the same subnet range as the test-container's subnet 10.0.3.0/24.  I'll explain later in this article, why the bridge has been assigned with an IP address.

Whenever LXC is created a default LXC bridge *lxcbr0 *is also created.  A Linux Bridge is a software equivalent of a physical switch used to interconnect physical host machines.  All containers spawned within a host would connect to this bridge by default.

The *veth3M3GH3* interface is a veth  ( virtual ethernet) type interface that corresponds to the *eth0* interface we saw inside the test-container.  The *veth3M3GH3* is connected automatically to the *lxcbr0 *by default when the LXC is spawned.

    $ brctl show
    bridge name bridge id       STP enabled interfaces
    lxcbr0      8000.fed9bb5ba681   no      veth3M3GH3
    $ 

## How LXC container talks to the outside world ?

First let's check if the connectivity between the container and the host machine is in place.  Try pinging the host machine's *eth0* interface IP address from inside the test-container.

    root@test-container:/# ping 192.168.85.132  -c 5
    PING 192.168.85.132 (192.168.85.132) 56(84) bytes of data.
    64 bytes from 192.168.85.132: icmp_seq=1 ttl=64 time=0.200 ms
    64 bytes from 192.168.85.132: icmp_seq=2 ttl=64 time=0.095 ms
    64 bytes from 192.168.85.132: icmp_seq=3 ttl=64 time=0.106 ms
    64 bytes from 192.168.85.132: icmp_seq=4 ttl=64 time=0.052 ms
    64 bytes from 192.168.85.132: icmp_seq=5 ttl=64 time=0.076 ms

    --- 192.168.85.132 ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 3999ms
    rtt min/avg/max/mdev = 0.052/0.105/0.200/0.052 ms
    root@test-container:/# 

Yey!! It works. But how does it work because these are two different subnets? Let's check the routing table of test-container.

    root@test-container:/# ip route
    default via 10.0.3.1 dev eth0 
    10.0.3.0/24 dev eth0  proto kernel  scope link  src 10.0.3.124 
    root@test-container:/# 

So looks like the *lxbr0*'s IP address(10.0.3.1) has been set as the default gateway to route any packet whose destination is other than 10.0.3.0/24 subnet.  As I described earlier, the *eth0* interface inside the container is the visible in the host network space as interface *veth3M3GH3.  *We know *veth3M3GH3 *is connected to *lxcbr0* by default.  This is how, the packets sent via the *eth0* interface in the container reaches the *lxcbr0* present outside the container in the host network space.

On a side note, one additional information we could glean from this output is that, the Linux Bridge *lxbr0* has been configured to function as a gateway or router in-addition to its bridge functionality.

Similarly, I was able to ping the container's *eth0* IP address from the host network space.

    $ ping 10.0.3.124 -c 5
    PING 10.0.3.124 (10.0.3.124) 56(84) bytes of data.
    64 bytes from 10.0.3.124: icmp_seq=1 ttl=64 time=0.208 ms
    64 bytes from 10.0.3.124: icmp_seq=2 ttl=64 time=0.068 ms
    64 bytes from 10.0.3.124: icmp_seq=3 ttl=64 time=0.062 ms
    64 bytes from 10.0.3.124: icmp_seq=4 ttl=64 time=0.095 ms
    64 bytes from 10.0.3.124: icmp_seq=5 ttl=64 time=0.076 ms

    --- 10.0.3.124 ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 3996ms
    rtt min/avg/max/mdev = 0.062/0.101/0.208/0.055 ms
    $

This is because, the host route table has been set up in such a way that any packet destined to 10.0.3.0/24 network is reachable via *lxcbr0*.

    $ ip route
    default via 192.168.85.2 dev eth0  proto static 
    10.0.3.0/24 dev lxcbr0  proto kernel  scope link  src 10.0.3.1 
    192.168.85.0/24 dev eth0  proto kernel  scope link  src 192.168.85.132  metric 1 
    $

## Inter-Networking Two LXC Containers:

I spawned another container named "*my-container*". It comes up with an IP of 10.0.3.8 on *eth0* interface.

    root@my-container:/# ifconfig
    eth0      Link encap:Ethernet  HWaddr 00:16:3e:22:5e:63  
              inet addr:10.0.3.8  Bcast:10.0.3.255  Mask:255.255.255.0
              inet6 addr: fe80::216:3eff:fe22:5e63/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:47 errors:0 dropped:0 overruns:0 frame:0
              TX packets:32 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000 
              RX bytes:8155 (8.1 KB)  TX bytes:3152 (3.1 KB)

    lo        Link encap:Local Loopback  
              inet addr:127.0.0.1  Mask:255.0.0.0
              inet6 addr: ::1/128 Scope:Host
              UP LOOPBACK RUNNING  MTU:65536  Metric:1
              RX packets:0 errors:0 dropped:0 overruns:0 frame:0
              TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:0 
              RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

Now check the routing table in *my-container*.

    root@my-container:/# ip route
    default via 10.0.3.1 dev eth0 
    10.0.3.0/24 dev eth0  proto kernel  scope link  src 10.0.3.8 
    root@my-container:/# 

On the host network space check if the equivalent virtual ethernet interface has been created.

    $ ifconfig
    eth0      Link encap:Ethernet  HWaddr 00:0c:29:9c:80:88  
              inet addr:192.168.85.132  Bcast:192.168.85.255  Mask:255.255.255.0
              inet6 addr: fe80::20c:29ff:fe9c:8088/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:353 errors:0 dropped:0 overruns:0 frame:0
              TX packets:798 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000 
              RX bytes:81156 (81.1 KB)  TX bytes:79957 (79.9 KB)

    lo        Link encap:Local Loopback  
              inet addr:127.0.0.1  Mask:255.0.0.0
              inet6 addr: ::1/128 Scope:Host
              UP LOOPBACK RUNNING  MTU:65536  Metric:1
              RX packets:301 errors:0 dropped:0 overruns:0 frame:0
              TX packets:301 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:0 
              RX bytes:45156 (45.1 KB)  TX bytes:45156 (45.1 KB)

    lxcbr0    Link encap:Ethernet  HWaddr fe:a9:0d:a9:aa:4d  
              inet addr:10.0.3.1  Bcast:10.0.3.255  Mask:255.255.255.0
              inet6 addr: fe80::744d:3ff:fef2:5c6e/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:233 errors:0 dropped:0 overruns:0 frame:0
              TX packets:266 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:0 
              RX bytes:19951 (19.9 KB)  TX bytes:36711 (36.7 KB)

    veth1TV1KE Link encap:Ethernet  HWaddr fe:a9:0d:a9:aa:4d  
              inet6 addr: fe80::fca9:dff:fea9:aa4d/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:48 errors:0 dropped:0 overruns:0 frame:0
              TX packets:67 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000 
              RX bytes:4592 (4.5 KB)  TX bytes:10023 (10.0 KB)

    veth3M3GH3 Link encap:Ethernet  HWaddr fe:d9:bb:5b:a6:81  
              inet6 addr: fe80::fcd9:bbff:fe5b:a681/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:91 errors:0 dropped:0 overruns:0 frame:0
              TX packets:129 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000 
              RX bytes:9221 (9.2 KB)  TX bytes:16408 (16.4 KB)

    $ 

So eth0 interface inside my-container is seen in the host network space as *veth1TV1KE* interface. Check if *veth1TV1KE* is connected to the Linux Bridge lxcbr0 by default.

    $ brctl show
    bridge name bridge id       STP enabled interfaces
    lxcbr0      8000.fea90da9aa4d   no      veth1TV1KE
                                veth3M3GH3
    $

Kewl! It is indeed connected to *lxcbr0*. Let's see if we could ping between the two containers.

    root@my-container:/# ping 10.0.3.124 -c 5
    PING 10.0.3.124 (10.0.3.124) 56(84) bytes of data.
    64 bytes from 10.0.3.124: icmp_seq=1 ttl=64 time=0.088 ms
    64 bytes from 10.0.3.124: icmp_seq=2 ttl=64 time=0.046 ms
    64 bytes from 10.0.3.124: icmp_seq=3 ttl=64 time=0.063 ms
    64 bytes from 10.0.3.124: icmp_seq=4 ttl=64 time=0.075 ms
    64 bytes from 10.0.3.124: icmp_seq=5 ttl=64 time=0.092 ms

    --- 10.0.3.124 ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 3997ms
    rtt min/avg/max/mdev = 0.046/0.072/0.092/0.020 ms
    root@my-container:/# 

Check the ping in the reverse direction.

    root@test-container:/# ping 10.0.3.8 -c 5
    PING 10.0.3.8 (10.0.3.8) 56(84) bytes of data.
    64 bytes from 10.0.3.8: icmp_seq=1 ttl=64 time=0.265 ms
    64 bytes from 10.0.3.8: icmp_seq=2 ttl=64 time=0.076 ms
    64 bytes from 10.0.3.8: icmp_seq=3 ttl=64 time=0.112 ms
    64 bytes from 10.0.3.8: icmp_seq=4 ttl=64 time=0.086 ms
    64 bytes from 10.0.3.8: icmp_seq=5 ttl=64 time=0.107 ms

    --- 10.0.3.8 ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 3999ms
    rtt min/avg/max/mdev = 0.076/0.129/0.265/0.069 ms

That's all folks !
