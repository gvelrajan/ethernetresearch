---
title: Linux Containers - Advanced LXC Networking - Part 01
description: ""
author: Ganesh Velrajan
tags: [
    Linux Containers, LXC, Linux Networking
]
date: 2016-11-19
categories: [
    GeekZone, Linux Containers, Networking
]
images: ["/images/linux/linux-container-lxc.jpeg"]
---

In the [previous article](https://ganeshtechblog.wordpress.com/2016/11/18/linux-container-lxc-networking-fundamentals/) I discussed about some basic LXC networking concepts. In this article I'll discuss about the LXC configuration files that play a key role in automated provisioning of the interfaces and bridges.

## Who Tells LXC to Create lxcbr0 ?

The answer is: */var/lib/lxc/test-container/config*.  The file has several sections and the networking section at the bottom of the file is what we are interested in.

    $ sudo cat /var/lib/lxc/test-container/config
    # Template used to create this container: /usr/share/lxc/templates/lxc-ubuntu
    # Parameters passed to the template:
    # For additional config options, please look at lxc.container.conf(5)

    # Common configuration
    lxc.include = /usr/share/lxc/config/ubuntu.common.conf

    # Container specific configuration
    lxc.rootfs = /var/lib/lxc/test-container/rootfs
    lxc.mount = /var/lib/lxc/test-container/fstab
    lxc.utsname = test-container
    lxc.arch = amd64

    # Network configuration
    lxc.network.type = veth
    lxc.network.flags = up
    lxc.network.link = lxcbr0
    lxc.network.hwaddr = 00:16:3e:b4:86:23

This is a container specific configuration file. There is a global or default configuration file at "*/etc/lxc/default.conf" *used by a container in the absence of a container specific configuration file.

    $ cat /etc/lxc/default.conf 
    lxc.network.type = veth
    lxc.network.link = lxcbr0
    lxc.network.flags = up
    lxc.network.hwaddr = 00:16:3e:xx:xx:xx

Who configures the IP address for the *lxcbr0* ? Who specifies the DHCP range for the containers connected to *lxcbr0* ? For this we need to look at file: */etc/init/lxc-net.conf*

    $ cat /etc/init/lxc-net.conf
    description "lxc network"
    author "Serge Hallyn <serge.hallyn@canonical.com>"

    start on starting lxc
    stop on stopped lxc

    env USE_LXC_BRIDGE="true"
    env LXC_BRIDGE="lxcbr0"
    env LXC_ADDR="10.0.3.1"
    env LXC_NETMASK="255.255.255.0"
    env LXC_NETWORK="10.0.3.0/24"
    env LXC_DHCP_RANGE="10.0.3.2,10.0.3.254"
    env LXC_DHCP_MAX="253"
    env LXC_DHCP_CONFILE=""
    env varrun="/run/lxc"
    env LXC_DOMAIN=""
    ...
    ...

In addition to the *lxcbr0* settings, the above file also programs the *iptables* with NAT rules(masquerade)such that the container can talk to the outside world or internet.

    $ sudo iptables --list -t nat
    Chain PREROUTING (policy ACCEPT)
    target     prot opt source               destination         

    Chain INPUT (policy ACCEPT)
    target     prot opt source               destination         

    Chain OUTPUT (policy ACCEPT)
    target     prot opt source               destination         

    Chain POSTROUTING (policy ACCEPT)
    target     prot opt source               destination         
    MASQUERADE  all  --  10.0.3.0/24         !10.0.3.0/24         
    $ 

## Interface Configuration Files:

The interface specific configurations are stored in */etc/network/interface*. You can edit this file to make changes to the interface configuration. In the example below, *eth0* is configured to use DHCP to talk to *lxcbr0, *acting as DHCP server, to get its IP address. However, if you want to assign a static IP address to the *test-container*, you may edit the file to do so. Changes to this file are carried across container reboot.

    root@test-container:/# cat /etc/network/interfaces
    # This file describes the network interfaces available on your system
    # and how to activate them. For more information, see interfaces(5).

    # The loopback network interface
    auto lo
    iface lo inet loopback

    auto eth0
    iface eth0 inet dhcp

To configure static IP address on *eth0* change the *eth0* configuration in the file to some like this:

    ...
    auto eth0
    iface eth0 inet static
    address 10.0.3.124
    gateway 10.0.3.1
    netmask 255.255.255.0
    ...
