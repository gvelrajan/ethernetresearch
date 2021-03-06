---
title: Linux Container Internals - Fundamentals of Linux Containers
description: ""
author: Ganesh Velrajan
tags: [
    Linux Containers, LXC, 
]
date: 2017-06-08
categories: [
    GeekZone, Linux Containers
]
images: ["/images/linux/linux-container-lxc.jpeg"]
---

## What are containers ?

Containers are groups of userspace processes running in an isolated or contained virtual environment of their own, created within a single host or a VM.  The isolated (contained) environment for each of the containers is created using the virtualization primitives provided by the modern Linux Kernel such as namespace, cgroups, etc.  Linux container management tools such as LXC and Docker are nothing but wrappers written on top these underlying Linux Kernel primitives to create and run containers.

Let’s get our hands dirty by playing with all these Linux Kernel primitives using some of the tools available in Linux.

To create a container we need a Linux container image first.

## What is a container image ?

A Linux container image is not any magic.  It’s just a plain rootfs tarball.  As long as you have a rootfs tarball you could spawn a container.

Please refer to this previous article on [how to build a rootfs tarball](http://35.238.255.76/geekzone/building-linux-rootfs-from-scratch/).  Alternatively, you could download the rootfs tarball from my git repo.  You could also potentially use "debootstap" to install a debian rootfs into a local directory.  It doesn't matter which method you choose to get a rootfs locally.

Once you have the rootfs tarball, you could untar it into a local directory as shown below.

    $ mktemp –d
    /tmp/tmp.3nfwC5jpyB
    $ mkdir /tmp/tmp.3nfwC5jpyB/rootfs
    $ sudo tar -xvf rootfs.tar -C /tmp/tmp.3nfwC5jpyB/rootfs/    # this command needs root permission

## Chroot:

The word "chroot" is a short form for "change the root filesystem" of the process.  Chroot command is a wrapper built on top of the chroot() system call.  By executing the command "chroot" we could change the rootfs of the current shell to start using the newly created rootfs tree as its root filesystem.  In other words, the “chroot” command changes the rootfs of the shell.  This command may require root privilege to execute.

    $ cd /tmp/ tmp.3nfwC5jpyB
    $ chroot rootfs/ /bin/sh
    chroot: cannot change root directory to rootfs/: Operation not permitted
    $ sudo chroot rootfs/ /bin/sh
    / #
    / # pwd
    /

Using the “chroot” command and the rootfs tarball we have created a new filesystem.  From now on, any command that you execute in the shell will be looked at in the new rootfs tree's bin or sbin folders and not from the parent rootfs tree's bin or sbin.

    / # which ls
    /bin/ls

This basically says the "ls" command was picked up from /tmp/tmp.3nfwC5jpyB/rootfs/bin and not from the parent rootfs /bin

    /# ls
    bin      lib      media    proc     sbin     usr
    dev      lib64    mnt      root     sys      var
    etc      linuxrc  opt      run      tmp

Now let's display the networking stuff from the shell using the "ifconfig" command.

    / # ifconfig
    ifconfig: /proc/net/dev: No such file or directory
    eth0      Link encap:Ethernet  HWaddr 00:0C:29:E7:B1:87
    inet addr:10.1.1.1  Bcast:10.1.1.255  Mask:255.255.255.0
    UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

    lo        Link encap:Local Loopback
    inet addr:127.0.0.1  Mask:255.0.0.0
    UP LOOPBACK RUNNING  MTU:65536  Metric:1

Why is the shell displaying the networking information from the parent networking namespace?

This is because the shell is still sharing the parent networking namespace. The chroot command just creates a new root filesystem.  It doesn’t change anything else for the shell, including the process namespace or the networking namespace.   We’ll discuss below in this article on how to create a new process namespace and a new networking namespace.

Now let’s exit from this new root filesystem and go back to the parent root filesystem.

    / # exit
    $
    $ pwd
    /tmp/tmp.3nfwC5jpyB
    $ ls /
    bin    dev   initrd.img  lost+found  opt   run   sys  var
    boot   etc   lib         media       proc  sbin  tmp  vmlinuz
    cdrom  home  lib64       mnt         root  srv   usr
    $
    $ ifconfig
    eth0 Link encap:Ethernet HWaddr 00:0c:29:e7:b1:87
    inet addr:10.1.1.1 Bcast:10.1.1.255 Mask:255.255.255.0
    inet6 addr: fd00::1:1/64 Scope:Link
    UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
    RX packets:19698 errors:0 dropped:0 overruns:0 frame:0
    TX packets:5089 errors:0 dropped:0 overruns:0 carrier:0
    collisions:0 txqueuelen:1000
    RX bytes:18775511 (18.7 MB) TX bytes:379475 (379.4 KB)

    lo Link encap:Local Loopback
    inet addr:127.0.0.1 Mask:255.0.0.0
    inet6 addr: ::1/128 Scope:Host
    UP LOOPBACK RUNNING MTU:65536 Metric:1
    RX packets:2252 errors:0 dropped:0 overruns:0 frame:0
    TX packets:2252 errors:0 dropped:0 overruns:0 carrier:0
    collisions:0 txqueuelen:0
    RX bytes:433390 (433.3 KB) TX bytes:433390 (433.3 KB)

## 

## Creating Process Namespace:

Containers are isolated groups of processes.  For process isolation to work, we need to create a separate process namespace for each of the containers.

Let's get our hands dirty further.  From the parent rootfs, create a new process and fetch its PID.

    $ top &
    [2] 7358
    $ ps | grep top
    7358 pts/15   00:00:00 top

Let’s “chroot” once again into the new rootfs.

    $ sudo chroot rootfs/ /bin/sh

Let's check if the “top” process is visible in the chroot’ed shell context. We need to mount the proc filesystem to /proc directory for “ps” command to work.

    / # mount -t proc proc /proc    
    / # ps | grep top
    7358 1000     top

Yes the process is visible in the chroot’ed shell context also.  This is because the shell is still sharing the process namespace of the parent.  To create a new process namespace for the shell, we need to use the command “**unshare**”.

“Unshare” command is a wrapper around the **unshare()** system call.

**[Note]{style="color: #0000ff;"}**:  **You need to have “util-linux” package version 2.23 or greater installed on your system for the unshare’s PID namespace feature to work**.

    $ unshare –fp /bin/bash

In the above command“-f” option specifies unshare to fork a new process to run /bin/bash.  And the “-p” option specifies unshare to create a new PID (process) namespace and a new process filesystem (**procfs**).

Now if you execute the “ps” command, as shown below, it will still display the processes running in the parent PID namespace.  This is because the /proc mount point from where the “ps” utility reads was mounted with the parent proc filesystem.

    $ ps -el
    F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
    4 S     0     1     0  0  80   0 - 48361 ep_pol ?        00:02:45 systemd
    1 S     0     2     0  0  80   0 -     0 kthrea ?        00:00:00 kthreadd
    1 S     0     3     2  0  80   0 -     0 smpboo ?        00:00:48 ksoftirqd/0
    1 S     0     4     2  0  80   0 -     0 worker ?        00:00:19 kworker/0:0
    1 S     0     5     2  0  60 -20 -     0 worker ?        00:00:00 kworker/0:0H
    1 S     0     8     2  0 -40   - -     0 smpboo ?        00:00:01 migration/0
    …

Now let’s manually mount the forked process’ procfs to /proc directory using the mount command.

    $ mount –t proc proc /proc
    $ ps -el
    F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
    4 S     0     1     0  0  80   0 - 29176 wait   pts/7    00:00:00 bash
    0 R     0    35     1  0  80   0 - 37226 -      pts/7    00:00:00 ps

If you notice the above output carefully, you’ll see that the bash thinks its PID 1.  Every newly created process namespace get its own numbering scheme, separate from the parent PID namespace.  This is main advantage of process or PID namespace, especially when moving containers from one host to another without having to worry about conflicting with the PID associated with existing processes in the new host.

The “unshare” command has an option to automatically mount the newly created procfs to the /proc directory using the “—mount-proc” option.

    $ ushare –fp –mount-proc /bin/bash
    $ ps -el
    F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
    4 S     0     1     0  1  80   0 - 29185 wait   pts/7    00:00:00 bash
    0 R     0    26     1  0  80   0 - 37226 -      pts/7    00:00:00 ps
    $

More interestingly, you could request “unshare” command to run any command after creating a new PID namespace, including “chroot” command.  So let’s see an example combining the two.

    $ sudo unshare -fp --mount-proc=$PWD/rootfs/proc/ chroot rootfs /bin/bash
    bash-4.3#
    bash-4.3# ps
    PID USER       VSZ STAT COMMAND
    1 root     13724 S    /bin/bash
    2 root     12000 R    ps
    bash-4.3# pwd
    /
    bash-4.3#

Let’s exit from the new rootfs and go back to the parent rootfs.

    bash-4.3# exit
    exit
    $ ps -el
    Error, do this: mount -t proc proc /proc
    $

There was an error because the procfs mounted in /proc was that of the forked process.  We need to remount the parent process’ procfs at /proc.

    $ mount -t proc proc /proc
    $ ps -el
    F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
    4 S     0     1     0  0  80   0 - 48361 ep_pol ?        00:02:45 systemd
    1 S     0     2     0  0  80   0 -     0 kthrea ?        00:00:00 kthreadd
    1 S     0     3     2  0  80   0 -     0 smpboo ?        00:00:48 ksoftirqd/0
    1 S     0     4     2  0  80   0 -     0 worker ?        00:00:19 kworker/0:0
    1 S     0     5     2  0  60 -20 -     0 worker ?        00:00:00 kworker/0:0H
    1 S     0     8     2  0 -40   - -     0 smpboo ?        00:00:01 migration/0
    …
    $

## 

## Creating Network Namespace:

Similar to process namespace, you could create a new network namespace using the "unshare" command with the “—net” option.  Here is an example.

    # From the default namespace
    $ ifconfig -s
    Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
    eth0      1350 108002738      0   2567 392    85862941      0      0      0 BMRU
    lo       65536  1406853      0      0 0       1406853      0      0      0 LRU
    $ unshare --net /bin/bash
    $ ifconfig         # inside the newly created network namespace
    $ brctl addbr bridge0
    $ ifconfig bridge0 up
    $ ifconfig
    bridge0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
    ether b6:ad:a5:9f:c5:9c  txqueuelen 0  (Ethernet)
    RX packets 0  bytes 0 (0.0 B)
    RX errors 0  dropped 0  overruns 0  frame 0
    TX packets 5  bytes 418 (418.0 B)
    TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    $

Now let’s exit the new network namespace and go back to the parent namespace which is the default namespace.

    $ exit
    exit
    $ ifconfig -s
    Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
    eth0      1350 108002738      0   2567 392    85862941      0      0      0 BMRU
    lo       65536  1406853      0      0 0       1406853      0      0      0 LRU
    $

We don’t see the bridge interface “bridge0” in the default namespace.  That explains the network namespace isolation.

An alternate way to create a network namespace is using the “ip netns” command.

    $ ip netns add newnet
    $ ip netns list
    newnet
    $

Now to enter into the “newnet” network namespace use the “exec” option followed by any command to run, as shown below.

    $ ip netns exec newnet bash
    $ ifconfig
    $ brctl addbr bridge0
    $ ifconfig bridge0 up
    $ ifconfig -s
    Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
    bridge0   1500        0      0      0 0             8      0      0      0 BMRU
    $ brctl show
    bridge name     bridge id               STP enabled     interfaces
    bridge0         8000.000000000000       no
    $ exit
    exit
    $ ifconfig -s
    Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
    eth0      1350 108002738      0   2567 392    85862941      0      0      0 BMRU
    lo       65536  1406853      0      0 0       1406853      0      0      0 LRU
    $

For each newly created network namespace, a file with the same name would be generated under the /var/run/netns folder.

    $ ls /var/run/netns/
    newnet
    $

### 

### Entering namespace using *setns()* and *nsenter*:

If you have a namespace created already but want to let some other process share the same namespace you could use the “nsenter” command.  “nsenter” command is just a wrapper written on top of the “setns()” systemcall.  “nsenter” command lets you to access any namespace not just network namespace.

    $ nsenter -h
    Usage:
    nsenter [options] [...]
    Run a program with namespaces of other processes.
    Options:
    -t, --target      target process to get namespaces from
    -m, --mount[=]   enter mount namespace
    -u, --uts[=]     enter UTS namespace (hostname etc)
    -i, --ipc[=]     enter System V IPC namespace
    -n, --net[=]     enter network namespace
    -p, --pid[=]     enter pid namespace
    -U, --user[=]    enter user namespace
    -S, --setuid      set uid in entered namespace
    -G, --setgid      set gid in entered namespace
    ...
    $

Here is an example C program to make a process to switch from default network namespace to "newnet" namespace using "setns()" systemcall.

    ...

    int main()
    {
            /* Spawn a child process */
            int id = fork();
            if ( id == 0) {
                /* Child Process still using default netns */
                system("ifconfig");
                /* Set the child to run in "newnet" */
                int fd = open("/var/run/netns/newnet", O_RDONLY);
                if (fd == -1) {
                    return -1;
                }

                if (setns(fd, 0) == -1) {
                    return (-2);
                }
            
                /* child now uses the "newnet" netns */
                system("ifconfig");
                ...

                return 0; 
            }
        /* parent using the default netns */
            system("ifconfig");
            ...
        return 0;
    }

In the example below, I’ll show how to access a network namespace using the “nsenter” command.

    $ nsenter --net=/var/run/netns/newnet /bin/bash
    $ ifconfig -s
    Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
    bridge0   1500        0      0      0 0             8      0      0      0 BMRU
    $ brctl show
    bridge name     bridge id               STP enabled     interfaces
    bridge0         8000.000000000000       no
    $ exit
    $

Now, wrapping up the network namespace section, let’s combine all the three commands explained above to create a container with its own rootfs, procfs and netns.

    $ pwd
    /tmp/tmp.3nfwC5jpyB
    $ ls
    rootfs
    $ sudo nsenter --net=/var/run/netns/newnet unshare -fp --mount-proc=rootfs/proc chroot rootfs/ /bin/bash
    bash-4.3# pwd
    /
    bash-4.3# ls
    bin         data        etc         lib         lost+found  mnt         proc        sbin        tftpboot    usr
    boot        dev         home        lib64       media       opt         run         sys         tmp         var
    bash-4.3# ifconfig
    bridge0   Link encap:Ethernet  HWaddr 7A:E8:FA:08:9F:0A
    UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
    RX packets:0 errors:0 dropped:0 overruns:0 frame:0
    TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
    collisions:0 txqueuelen:0
    RX bytes:0 (0.0 B)  TX bytes:648 (648.0 B)
    bash-4.3# ps
    PID USER       VSZ STAT COMMAND
    1 root     13724 S    /bin/bash
    4 root     12000 R    ps
    bash-4.3# exit
    exit
    $ pwd
    /tmp/tmp.3nfwC5jpyB
    $ ifconfig -s
    Iface   MTU Met   RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
    eth0       1500 0    294377      0      0 0        146774      0      0      0 BMRU
    lo        65536 0      4795      0      0 0          4795      0      0      0 LRU
    $

## cgroups:

“cgroups” is a short form for “control groups”.  cgroups provide us the knobs to control and ration the system resources utilized by containers.  Without the cgroups support in the Linux Kernel, containers could contend with other processes for system resources such as CPU, memory, network etc., and could potentially starve out other processes or containers running in the same host.

The kernel exposes the cgroups through the **/sys/fs/cgroup** directory.

    $ sudo su
    # ls /sys/fs/cgroup/
    blkio/      cpuacct/    devices/    hugetlb/    net_cls/    perf_event/
    cpu/        cpuset/     freezer/    memory/     net_prio/   systemd/

For this example, let’s restrict the CPU resource used by a process.  A new cgroup can be easily created by just adding a directory under /sys/fs/cgroup/cpu/.  For example, I added “demo” cpu cgroup under /sys/fs/cgroup/cpu/.

    # mkdir /sys/fs/cgroup/cpu/demo

Once we create a directory under a cgroup category, the kernel automatically fills the directory with files necessary to manage the cgroup.

    # ls –l /sys/fs/cgroup/cpu/demo/
    total 0
    -rw-r--r-- 1 root root 0 Jun  9 20:58 cgroup.clone_children
    -rw-r--r-- 1 root root 0 Jun  9 20:58 cgroup.procs
    -rw-r--r-- 1 root root 0 Jun  9 21:17 cpu.cfs_period_us
    -rw-r--r-- 1 root root 0 Jun  9 21:18 cpu.cfs_quota_us
    -rw-r--r-- 1 root root 0 Jun  9 21:09 cpu.shares
    -r--r--r-- 1 root root 0 Jun  9 20:58 cpu.stat
    -rw-r--r-- 1 root root 0 Jun  9 20:58 notify_on_release
    -rw-r--r-- 1 root root 0 Jun  9 20:59 tasks
    #

Each file listed above represents a configurable parameter for the cpu cgroup.  You could dump the content of these files to see the default values.

    # cat /sys/fs/cgroup/cpu/demo/cpu.shares
    1024

Now let’s configure the “demo” cpu cgroup with cpu.shares set to 25% such that any processes or containers using the cgroup would be restricted to use just 25% of the available 100% cpu cycles.  This restriction would come into play only when there is a contention for the cpu.  Meaning, if there are no other processes contending for the cpu, then the process assigned to the “demo” cpu group could potentially use upto 100% of the available cpu.

    # echo 256 > /sys/fs/cgroup/cpu/demo/cpu.shares
    # cat /sys/fs/cgroup/cpu/demo/cpu.shares
    256

To make a process to join the cgroup, just write the PID of the process to **/sys/fs/cgroup/cpu/demo/tasks** file.  In the example below, I added the PID of the current shell to the “tasks” file.

    # echo $$ > /sys/fs/cgroup/cpu/demo/tasks

Here is a simple C program with an infinite loop to use 100% cpu.

    // task.c
    #include "stdio.h"
    main()
    {
       while(1);
    }

Compile and run the program.

    # gcc task.c –o task1
    # cp task1 task2

Run “task1” in the current shell.

    # ./task1

Now open another shell window and run the “top” command to monitor the CPU utilization by “task1”.

    $ top

    top - 22:23:34 up 1 day,  5:50,  5 users,  load average: 0.89, 0.52, 0.99
    Tasks: 235 total,   3 running, 231 sleeping,   1 stopped,   0 zombie
    %Cpu(s):100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
    KiB Mem:   2032384 total,  1716488 used,   315896 free,   109260 buffers
    KiB Swap:  2094076 total,     5400 used,  2088676 free.   724224 cached Mem 

    PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
    19420 root      20   0    4196    628    548 R 99.1  0.0   2:00.22 task1      
    228 root      20   0       0      0      0 S  0.3  0.0   0:04.76 jbd2/sda1-8
    1096 jenkins   20   0 1622688 228188  21648 S  0.3 11.2   3:44.05 java
    1 root      20   0   33900   4344   2712 S  0.0  0.2   0:02.46 init
    $

It shows almost 100% CPU utilization by task1.  This is because no other process is contending with task1 for the cpu resource. Now let’s open another shell window and run “task2” in it.

    $ ./task2

Now look at the window where the “top” command is still running.  It would show that as task2 cranks up on the cpu utilization, task1’s cpu utilization comes down close to the configured CPU utilization limit.

    $ top

    top - 22:26:44 up 1 day,  5:53,  5 users,  load average: 1.56, 0.93, 1.06
    Tasks: 238 total,   3 running, 234 sleeping,   1 stopped,   0 zombie
    %Cpu(s):100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
    KiB Mem:   2032384 total,  1718652 used,   313732 free,   109308 buffers
    KiB Swap:  2094076 total,     5400 used,  2088676 free.   724232 cached Mem 

    PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
    19442 root      20   0    4196    636    556 R 79.3  0.0   0:12.64 task2      
    19420 root      20   0    4196    628    548 R 19.9  0.0   4:15.70 task1      
    3201 www-data  20   0  360480   6176   2628 S  0.7  0.3   0:45.12 apache2
    1 root      20   0   33900   4344   2712 S  0.0  0.2   0:02.46 init
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kthreadd

When you want to delete a cgroup, execute the “rmdir” command to remove the directory.  Before you delete a cgroup make sure all the processes using the cpu have exited.

    # exit
    $ sudo rmdir /sys/fs/cgroup/cpu/demo/

Similarly, you could manage the memory cgroup by creating a new folder under **/sys/fs/cgroup/memory** and configure the appropriate files under it.

## Conclusion:

**Containers are no magic**.  It basically leverages some of the Linux Kernel primitives such as namespace, chroot, cgroups etc to create a controlled, isolated, pseudo-virtual environment for a group of processes to run.

Congratulations! Now that you have learnt these container secrets, you have become an expert in container technology!!
