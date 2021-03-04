---
title: Linux Networking Commands to Debug Troubleshoot IP / UDP / TCP Packet Loss
description: ""
author: Ganesh Velrajan
tags: [
    Linux Networking, TCP, UDP, IP, Packet Loss
]
date: 2017-10-13
categories: [
    GeekZone, Linux Containers, Networking
]
images: ["/images/linux/programming.jpg"]
---

Recently, I happened to debug an UDP flow packet loss issue happening within an ubuntu LXC container.  It was not obviously evident why the UDP packets were getting dropped.  However, the *tcpdump* utility showed that the packets are indeed being received by the ingress interface (eth0) of the container.  I executed a bunch of Linux networking related commands and finally figured out why and where the packets were getting dropped.

In this article, I'll share a comprehensive list of the Linux networking commands and debugging tools I used to debug the UDP packet loss problem.

## How to check debug troubleshoot packet loss

Here is the step by step instructions on how to debug check troubleshoot a network packet loss in linux.

### tcpdump and wireshark:

The very first step in any packet debugging process is to capture the packet entering or leaving the NIC using the "*tcpdump*" utility.  You could potentially save the *tcpdump* captured packets in a file and open it using the "*wireshark*" GUI tool.  In the GUI, glance over the various packet fields such as Source MAC, Destination MAC, Source IP, Destination IP, TTL, Checksum etc. and ensure they are all correct.

    $ tcpdump -i eth0 -c 1 -w mypacket.pcap 

    tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
    1 packet captured
    87 packets received by filter
    0 packets dropped by kernel

    $ wireshark mypacket.pcap   # this will open the GUI

## Interface Statistics:

Once you confirmed that the packets have entered through the NIC and they are not malformed packets, move on to the next step, which is to look at the network interface statistics and ensure they don't have any drops or errors.

Here are some of the commands you could use to do so.

    user@ubuntu:~$ ifconfig -a
    eth0 Link encap:Ethernet HWaddr 00:0c:29:e7:b1:87 
     inet addr:192.168.85.134 Bcast:192.168.85.255 Mask:255.255.255.0
     inet6 addr: fe80::20c:29ff:fee7:b187/64 Scope:Link
     UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
     RX packets:8236 errors:0 dropped:0 overruns:0 frame:0
     TX packets:3390 errors:0 dropped:0 overruns:0 carrier:0
     collisions:0 txqueuelen:1000 
     RX bytes:9677592 (9.6 MB) TX bytes:294562 (294.5 KB)

    lo Link encap:Local Loopback 
     inet addr:127.0.0.1 Mask:255.0.0.0
     inet6 addr: ::1/128 Scope:Host
     UP LOOPBACK RUNNING MTU:65536 Metric:1
     RX packets:4043 errors:0 dropped:0 overruns:0 frame:0
     TX packets:4043 errors:0 dropped:0 overruns:0 carrier:0
     collisions:0 txqueuelen:0 
     RX bytes:769351 (769.3 KB) TX bytes:769351 (769.3 KB)

    user@ubuntu:~$

You could also look at the interface statistics file in the */sys* filesystem.

    user@ubuntu:~$ grep '.*' /sys/class/net/eth0/statistics/* 
    /sys/class/net/eth0/statistics/collisions:0
    /sys/class/net/eth0/statistics/multicast:0
    /sys/class/net/eth0/statistics/rx_bytes:9380575
    /sys/class/net/eth0/statistics/rx_compressed:0
    /sys/class/net/eth0/statistics/rx_crc_errors:0
    /sys/class/net/eth0/statistics/rx_dropped:0
    /sys/class/net/eth0/statistics/rx_errors:0
    /sys/class/net/eth0/statistics/rx_fifo_errors:0
    /sys/class/net/eth0/statistics/rx_frame_errors:0
    /sys/class/net/eth0/statistics/rx_length_errors:0
    /sys/class/net/eth0/statistics/rx_missed_errors:0
    /sys/class/net/eth0/statistics/rx_over_errors:0
    /sys/class/net/eth0/statistics/rx_packets:7954
    /sys/class/net/eth0/statistics/tx_aborted_errors:0
    /sys/class/net/eth0/statistics/tx_bytes:278911
    /sys/class/net/eth0/statistics/tx_carrier_errors:0
    /sys/class/net/eth0/statistics/tx_compressed:0
    /sys/class/net/eth0/statistics/tx_dropped:0
    /sys/class/net/eth0/statistics/tx_errors:0
    /sys/class/net/eth0/statistics/tx_fifo_errors:0
    /sys/class/net/eth0/statistics/tx_heartbeat_errors:0
    /sys/class/net/eth0/statistics/tx_packets:3166
    /sys/class/net/eth0/statistics/tx_window_errors:0

There is a tool named "*ethtool*" which you could download and install.  It also would display the above information more elegantly.

    user@ubuntu:~$ ethtool -S eth0
    NIC statistics:
     rx_packets: 7958
     tx_packets: 3168
     rx_bytes: 9413211
     tx_bytes: 279313
     rx_broadcast: 0
     tx_broadcast: 0
     rx_multicast: 0
     tx_multicast: 0
     rx_errors: 0
     tx_errors: 0
     tx_dropped: 0
     multicast: 0
     collisions: 0
     rx_length_errors: 0
     rx_over_errors: 0
     rx_crc_errors: 0
     rx_frame_errors: 0
     rx_no_buffer_count: 0
     rx_missed_errors: 0
     tx_aborted_errors: 0
     tx_carrier_errors: 0
     tx_fifo_errors: 0
     tx_heartbeat_errors: 0
     tx_window_errors: 0
     tx_abort_late_coll: 0
     tx_deferred_ok: 0
     tx_single_coll_ok: 0
     tx_multi_coll_ok: 0
     tx_timeout_count: 0
     tx_restart_queue: 0
     rx_long_length_errors: 0
     rx_short_length_errors: 0
     rx_align_errors: 0
     tx_tcp_seg_good: 0
     tx_tcp_seg_failed: 0
     rx_flow_control_xon: 0
     rx_flow_control_xoff: 0
     tx_flow_control_xon: 0
     tx_flow_control_xoff: 0
     rx_long_byte_count: 9413211
     rx_csum_offload_good: 7085
     rx_csum_offload_errors: 0
     alloc_rx_buff_failed: 0
     tx_smbus: 0
     rx_smbus: 0
     dropped_smbus: 0
    user@ubuntu:~$

You could also use the "*netstat*" command with "-i" option to display the interface statistics.

    user@ubuntu:~$ netstat -i
    Kernel Interface table
    Iface   MTU Met   RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
    eth0       1500 0      7954      0      0 0          3166      0      0      0 BMRU
    lo        65536 0      3807      0      0 0          3807      0      0      0 LRU
    user@ubuntu:~$

## Try Raspberry Pi Remote SSH Access for Free From SocketXP

### [IoT Remote SSH or Raspberry Pi Remote SSH Access](https://www.socketxp.com/iot/raspberry-pi-remote-ssh-access/)

## IP Filters:

After the packet enters the box and doesn't get dropped at the NIC layer, they are subject to IP ingress filters.  We can dump the IP filters table to see if there are any filters configured to drop(REJECT) the packet and the corresponding dropped packet count for each filter configured in the table.

    user@ubuntu:~$ sudo iptables -vL
     [sudo] password for user: 
     Chain INPUT (policy ACCEPT 7408 packets, 9828K bytes)
     pkts bytes target prot opt in out source destination 
     0 0 ACCEPT udp -- virbr0 any anywhere anywhere udp dpt:domain
     0 0 ACCEPT tcp -- virbr0 any anywhere anywhere tcp dpt:domain
     0 0 ACCEPT udp -- virbr0 any anywhere anywhere udp dpt:bootps
     0 0 ACCEPT tcp -- virbr0 any anywhere anywhere tcp dpt:bootps
     
     Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
     pkts bytes target prot opt in out source destination 
     0 0 ACCEPT all -- virbr0 virbr0 anywhere anywhere 
     0 0 REJECT all -- any virbr0 anywhere anywhere reject-with icmp-port-unreachable
     0 0 REJECT all -- virbr0 any anywhere anywhere reject-with icmp-port-unreachable
     
     Chain OUTPUT (policy ACCEPT 6671 packets, 920K bytes)
     pkts bytes target prot opt in out source destination 
     0 0 ACCEPT udp -- any virbr0 anywhere anywhere udp dpt:bootpc
     user@ubuntu:~$

## Ethernet Header and IP Header Processing:

Any packet that is not dropped by the filters would go through the different layers in the TCP/IP stack for further processing.   First, it goes through the L2 layer which is the Ethernet layer for Ethernet Header processing.  Primarily it checks if the destination MAC is its own MAC address or someone else and makes a decision to forward or consume the packet or pass on to the next layer in the TCP/IP stack.  It is in this layer that the source MAC learning happens.  Meaning, the source MAC can be reached through the incoming interface and it maps to the Source IP in the packet (ARP resolution).

In the L3 layer processing, IP header fields such as *destination* *IP*, *TTL* etc. will be looked at.  If the destination IP is its own IP, then the packet would be consumed or sent to the upper layers(UDP/TCP) for further processing.  If the destination IP  is not the box's IP address, then a routing table lookup will be done to find the gateway or the next-hop to send the packet to.

### Displaying Routing Table:

Check the *routing table* to see if the routes are present.  In the example below, we show an IPv6 routing table ( *--inet6* signifies that ).

    user@ubuntu:/# netstat -r --inet6

    Kernel IPv6 routing table
    Destination                    Next Hop                   Flag Met Ref Use If
    fe80::/64                      ::                         U    256 0     0 eth0
    ::/0                           ::                         !n   -1  1    37 lo
    ::1/128                        ::                         Un   0   1     2 lo
    fe80::216:3eff:fe88:c1d3/128   ::                         Un   0   1     0 lo
    ff00::/8                       ::                         U    256 0     0 eth0
    ::/0                           ::                         !n   -1  1    37 lo
    user@ubuntu:/#

### **Displaying Arp Table:**

Make sure the arp for the next hop router is resolved and is learnt via correct interface.

    user@ubuntu:~$ arp
    Address                  HWtype  HWaddress           Flags Mask            Iface
    192.168.85.2             ether   00:50:56:f5:0c:c4   C                     eth0
    192.168.85.254           ether   00:50:56:e3:b6:fc   C                     eth0
    user@ubuntu:~$

## Socket Connection State:

You should next check if the UDP/TCP socket is in the correct state.  You could again use the "*netstat*" command with appropriate options to display the same, as shown below.

    user@ubuntu:~$ netstat --all --udp  --tcp
    Active Internet connections (servers and established)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State     
    tcp        0      0 ubuntu:domain           *:*                     LISTEN    
    tcp        0      0 *:ssh                   *:*                     LISTEN    
    tcp        0      0 localhost:ipp           *:*                     LISTEN    
    tcp        0      0 *:git                   *:*                     LISTEN    
    tcp6       0      0 [::]:ssh                [::]:*                  LISTEN    
    tcp6       0      0 ip6-localhost:ipp       [::]:*                  LISTEN    
    udp        0      0 *:52707                 *:*                               
    udp        0      0 *:ipp                   *:*                               
    udp6       0      0 [::]:53788              [::]:*                            
    udp6       0      0 [::]:36751              [::]:*                            

After you ensure that the socket state is correct, you can list the IP/UDP/TCP packet statistics using the following netstat statistics ( -s option ) command.

## Netstat Statistics:

    user@ubuntu:~$ netstat -s
    Ip:
        7499 total packets received
        3 with invalid addresses
        0 forwarded
        0 incoming packets discarded
        7463 incoming packets delivered
        6731 requests sent out
        8 outgoing packets dropped

    Icmp:
        22 ICMP messages received
        0 input ICMP message failed.
        ICMP input histogram:
            destination unreachable: 20
            echo requests: 2
        22 ICMP messages sent
        0 ICMP messages failed
        ICMP output histogram:
            destination unreachable: 20
            echo replies: 2
    IcmpMsg:
            InType3: 20
            InType8: 2
            OutType0: 2
            OutType3: 20
    Tcp:
        155 active connections openings
        77 passive connection openings
        60 failed connection attempts
        0 connection resets received
        0 connections established
        6691 segments received
        6114 segments send out
        0 segments retransmited
        0 bad segments received.
        60 resets sent
    Udp:
        785 packets received
        17 packets to unknown port received.
        0 packet receive errors
        666 packets sent
        IgnoredMulti: 216
    UdpLite:
    TcpExt:
        91 TCP sockets finished time wait in fast timer
        192 delayed acks sent
        Quick ack mode was activated 1 times
        315 packets directly queued to recvmsg prequeue.
        48452 bytes directly in process context from backlog
        939769 bytes directly received in process context from prequeue
        3004 packet headers predicted
        360 packets header predicted and directly queued to user
        604 acknowledgments not containing data payload received
        792 predicted acknowledgments
        TCPLossProbes: 1
        IPReversePathFilter: 10
        TCPRcvCoalesce: 1012
        TCPOrigDataSent: 2429
    IpExt:
        InNoRoutes: 23
        InMcastPkts: 266
        OutMcastPkts: 243
        InBcastPkts: 216
        InOctets: 9844085
        OutOctets: 926822
        InMcastOctets: 35621
        OutMcastOctets: 30950
        InBcastOctets: 24635
        InNoECTPkts: 11099
    user@ubuntu:~$

## Dumping SNMP Counters:

You could also dump the SNMP Counters to where exactly the packet is being dropped and for what exact reason.

    user@ubuntu:~$ cat /proc/net/snmp6
    Ip6InReceives                        221
    Ip6InHdrErrors                       0
    Ip6InTooBigErrors                    0
    Ip6InNoRoutes                        0
    Ip6InAddrErrors                      0
    Ip6InUnknownProtos                   0
    Ip6InTruncatedPkts                   0
    Ip6InDiscards                        0
    Ip6InDelivers                        221
    Ip6OutForwDatagrams                  0
    Ip6OutRequests                       277
    Ip6OutDiscards                       2
    Ip6OutNoRoutes                       39
    Ip6ReasmTimeout                      0
    Ip6ReasmReqds                        0
    Ip6ReasmOKs                          0
    Ip6ReasmFails                        0
    Ip6FragOKs                           0
    Ip6FragFails                         0
    Ip6FragCreates                       0
    Ip6InMcastPkts                       144
    Ip6OutMcastPkts                      239
    Ip6InOctets                          29087
    Ip6OutOctets                         33147
    Ip6InMcastOctets                     22382
    Ip6OutMcastOctets                    29666
    Ip6InBcastOctets                     0
    Ip6OutBcastOctets                    0
    Ip6InNoECTPkts                       221
    Ip6InECT1Pkts                        0
    Ip6InECT0Pkts                        0
    Ip6InCEPkts                          0
    Icmp6InMsgs                          2
    Icmp6InErrors                        0
    Icmp6OutMsgs                         61
    Icmp6OutErrors                       0
    Icmp6InCsumErrors                    0
    Icmp6InDestUnreachs                  0
    Icmp6InPktTooBigs                    0
    Icmp6InTimeExcds                     0
    Icmp6InParmProblems                  0
    Icmp6InEchos                         0
    Icmp6InEchoReplies                   0
    Icmp6InGroupMembQueries              0
    Icmp6InGroupMembResponses            0
    Icmp6InGroupMembReductions           0
    Icmp6InRouterSolicits                0
    Icmp6InRouterAdvertisements          0
    Icmp6InNeighborSolicits              0
    Icmp6InNeighborAdvertisements        2
    Icmp6InRedirects                     0
    Icmp6InMLDv2Reports                  0
    Icmp6OutDestUnreachs                 0
    Icmp6OutPktTooBigs                   0
    Icmp6OutTimeExcds                    0
    Icmp6OutParmProblems                 0
    Icmp6OutEchos                        0
    Icmp6OutEchoReplies                  0
    Icmp6OutGroupMembQueries             0
    Icmp6OutGroupMembResponses           0
    Icmp6OutGroupMembReductions          0
    Icmp6OutRouterSolicits               15
    Icmp6OutRouterAdvertisements         0
    Icmp6OutNeighborSolicits             6
    Icmp6OutNeighborAdvertisements       0
    Icmp6OutRedirects                    0
    Icmp6OutMLDv2Reports                 40
    Icmp6InType136                       2
    Icmp6OutType133                      15
    Icmp6OutType135                      6
    Icmp6OutType143                      40
    Udp6InDatagrams                      245
    Udp6NoPorts                          0
    Udp6InErrors                         0
    Udp6OutDatagrams                     136
    Udp6RcvbufErrors                     0
    Udp6SndbufErrors                     0
    Udp6InCsumErrors                     0
    Udp6IgnoredMulti                     0
    UdpLite6InDatagrams                  0
    UdpLite6NoPorts                      0
    UdpLite6InErrors                     0
    UdpLite6OutDatagrams                 0
    UdpLite6RcvbufErrors                 0
    UdpLite6SndbufErrors                 0
    UdpLite6InCsumErrors                 0
    user@ubuntu:~$

## Miscellaneous Items:

### Displaying Bridges:

Make sure the bridges are configured and connected to the interfaces correctly.  If not the packets may not be bridged out on the correct interface.

    user@ubuntu:~$ brctl show
    bridge name     bridge id       STP enabled     interfaces
    lxcbr0          8000.000000000000    no        vethI6Q4U6
    virbr0          8000.000000000000    yes       

    user@ubuntu:~$

 

***Sometimes an entire*** ***networking functionality or feature may be disabled in the Linux networking stack*.**  For example, the following command shows you how to check if IPv6 functionality is enabled or disabled in the Linux kernel.  If disabled, IPv6 packet may be dropped silently.  These settings can be enabled or disabled on a global basis, as shown below:

    user@ubuntu:~$ sysctl net.ipv6.conf.all.disable_ipv6
    net.ipv6.conf.all.disable_ipv6 = 0
    user@ubuntu:~$ sysctl net.ipv6.conf.default.disable_ipv6
    net.ipv6.conf.default.disable_ipv6 = 0

or on a per interface basis, as show below:

    user@ubuntu:~$ sysctl net.ipv6.conf.lo.disable_ipv6
    net.ipv6.conf.lo.disable_ipv6 = 0
    user@ubuntu:~$ sysctl net.ipv6.conf.eth0.disable_ipv6 
    net.ipv6.conf.eth0.disable_ipv6 = 0
    user@ubuntu:~$

Alternatively, you could refer to the */etc/sysctl.conf* file for these settings. In the below example, the settings are commented out.

    user@ubuntu:~$ cat /etc/sysctl.conf | grep ipv6
    #net.ipv6.conf.all.forwarding=1
    #net.ipv6.conf.all.accept_redirects = 0
    #net.ipv6.conf.all.accept_source_route = 0
    user@ubuntu:~$

That's all folks !

Interested in learning Docker Containers and Kubernetes Devops from a professional and experienced instructor?

## Find online courses from author Ganesh Velrajan here.

### [Docker and Kubernetes Courses in Udemy at Author Discount Rates.](/online-courses-training/)

  
