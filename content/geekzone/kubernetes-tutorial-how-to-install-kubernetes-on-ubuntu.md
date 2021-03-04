---
title: Kubernetes Tutorial - How to Install Kubernetes on Ubuntu Baremetal
description: ""
author: Ganesh Velrajan
tags: [
    Container, Docker, Kubernetes
]
date: 2018-07-19
categories: [
    GeekZone, Kubernetes
]
images: ["/images/kubernetes/kubernetes.jpg"]
---

The primary goal of this tutorial is teach how to install Kubernetes on Ubuntu and configure a Kubernetes Cluster using a Master and Worker node.

\[This is the the first lab in the Kubernetes Tutorial.\]

The Ubuntu server can be a baremetal Ubuntu server or an Ubuntu VM.  The steps involved are exactly the same.

Here is the agenda for in this lab.

1.  Bring up two Ubuntu 16.4 VMs.
2.  Install and bring up Kubernetes cluster master node,
3.  Install and bring up Kubernetes cluster worker node,
4.  Install and bring up Calico CNI.
5.  Download and run NGINX container.
6.  Verify NGINX pod runs in the cluster.

## Google Cloud Platform(GCP):

I signed up for Google Cloud Platform as it provides a \$300 worth of cloud usage for 1 year free.  I you prefer you can also sign up and run this tutorial in GCP.

Alternatively, you could run the VMs on your laptop.

I spawned two Ubuntu 16.4 VM's.

[***Please ensure that you use Ubuntu 16.4 for this tutorial.  I tried on 18.4 but I faced several challenges and open bugs in Kubernetes.***]{style="text-decoration: underline; color: #ff0000;"}

[***Please ensure that each VM has at least 2 Cores and 4GB RAM.***]{style="text-decoration: underline; color: #ff0000;"}

## Installing Kubernetes on Ubuntu:

The installation procedure is common for both master and worker nodes in a Kubernetes cluster.  Follow the procedure below to install Kubernetes in both the Ubuntu VM's you have spawned.

    Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.13.0-1019-gcp x86_64)

    * Documentation: https://help.ubuntu.com
     * Management: https://landscape.canonical.com
     * Support: https://ubuntu.com/advantage

    Get cloud support with Ubuntu Advantage Cloud Guest:
     http://www.ubuntu.com/business/services/cloud

    0 packages can be updated.
    0 updates are security updates.




    Last login: Sun Jul 8 04:42:10 2018 from 173.194.93.35
    master:~$ ifconfig
    ens4 Link encap:Ethernet HWaddr 42:01:0a:a0:00:02 
     inet addr:10.160.0.2 Bcast:10.160.0.2 Mask:255.255.255.255
     inet6 addr: fe80::4001:aff:fea0:2/64 Scope:Link
     UP BROADCAST RUNNING MULTICAST MTU:1460 Metric:1
     RX packets:547 errors:0 dropped:0 overruns:0 frame:0
     TX packets:499 errors:0 dropped:0 overruns:0 carrier:0
     collisions:0 txqueuelen:1000 
     RX bytes:413630 (413.6 KB) TX bytes:61585 (61.5 KB)

    lo Link encap:Local Loopback 
     inet addr:127.0.0.1 Mask:255.0.0.0
     inet6 addr: ::1/128 Scope:Host
     UP LOOPBACK RUNNING MTU:65536 Metric:1
     RX packets:0 errors:0 dropped:0 overruns:0 frame:0
     TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
     collisions:0 txqueuelen:1000 
     RX bytes:0 (0.0 B) TX bytes:0 (0.0 B)

Check if the worker node can be pinged from the master node.  This is key to forming the Kubernetes cluster.

    master:~$ ping 10.160.0.3
    PING 10.160.0.3 (10.160.0.3) 56(84) bytes of data.
    64 bytes from 10.160.0.3: icmp_seq=1 ttl=64 time=1.48 ms
    64 bytes from 10.160.0.3: icmp_seq=2 ttl=64 time=0.273 ms
    ^C
    --- 10.160.0.3 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 1001ms
    rtt min/avg/max/mdev = 0.273/0.877/1.481/0.604 ms

### Docker Installation:

We need to install Docker daemon if we need to use Docker images to run Docker containers.  Remember from previous tutorials that Kubernetes is just an orchestration platform.  It manages Docker and LXC containers.

    master:~$ sudo apt-get update   
   && sudo apt-get install -qy docker.io

### Setup Kubernetes apt repository:

First we need to download the Kubernetes apt packages from the Kubernetes web site to a local repository.

    master:~$ sudo apt-get update   
   && sudo apt-get install -y apt-transport-https   
   && curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -


    master-1:~$ echo "deb http://apt.kubernetes.io/ kubernetes-xenial main"   
   | sudo tee -a /etc/apt/sources.list.d/kubernetes.list   
   && sudo apt-get update

### Install Kubernetes

Next install the required Kubernetes packages from the local repo.

    master:~$ sudo apt-get update   
   && sudo apt-get install -y   
   kubelet   
   kubeadm   
   kubernetes-cni

This completes the installation of all the softwares required to setup Kubernetes Cluster and run Docker containers in Kubernetes.

### Simple Installation Script

For ease of use, I have captured all the commands used so far in a shell script file named **[kube-install.sh](http://ethernetresearch.com/ganesh/kube-install.sh)**.  All you need to do is simply download the shell script and execute "**sh  kube-install.sh**" as a non-root user.  It will install all the software for you in one shot, without requiring any user intervention.

Please make sure that the Linux swap is turn off.  Kubernetes requires that all pods/containers always stays in memory and not swapped to the disk.

    master:~$ sudo apt install mount   #install swapoff in mount pkg
    master:~$ sudo swapoff --all
    master:~$ cat /proc/swaps
    Filename Type Size Used Priority

> *I hope you followed all the above set of steps to install Docker and Kubernetes on the worker node also.  If not, please complete the Kubernetes installation in the worker node before we move on the the next steps.*

## Inialize the master node:

From now on, we execute and manage the clusters and pods from the master node only.  Worker nodes don't can't be used for cluster and pod management.

Initialalize the master node with the following command.  The default subnet for Calico CNI is 192.168.0.0/16 and Flannel CNI is 10.244.0.0/16.  We'll be using Calico CNI plugin for this example.  Hence we'll set "—pod-network-cidr" to the Calico subnet.

Set the "—apiserver-advertise-address" with the ip address of the master node.

    master:~$ sudo kubeadm init —pod-network-cidr=192.168.0.0/16 —apiserver-advertise-address=10.160.0.2

    ...

    Your Kubernetes master has initialized successfully!

    To start using your cluster, you need to run the following as a regular user:

    mkdir -p $HOME/.kube
     sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
     sudo chown $(id -u):$(id -g) $HOME/.kube/config

    You should now deploy a pod network to the cluster.
    Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
     https://kubernetes.io/docs/concepts/cluster-administration/addons/

    You can now join any number of machines by running the following on each node
    as root:

    kubeadm join 10.160.0.2:6443 --token dg6zhb.naxuxb5wd78tdsqz --discovery-token-ca-cert-hash sha256:af9e87508dc6c2c52b6f51dd88635f2dc9f061dce27368f3ac9913b52522cac3

### 

## Setup Container Networking Interface(CNI)

As the init logs above says, we need to install and setup the CNI. We'll use the Calico CNI plugin in this example.  We should provide the calico.yaml file from the calico's project website.

    master:~$ kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

Usually Pods or Containers are not run in the master node. because master nodes are used for controlling purposes and not for executing the jobs.  But, there is an option to override this to run a single node cluster with just the master node.  The pods will run in the master node itself.   Here is the command to make master node to run the Pods.

    $ kubectl taint nodes --all node-role.kubernetes.io/master-

However, if you want to build a real world cluster, then skip the above command and go add the worker nodes as explained below.

## Configure the Worker:

We need to use the token generated by the master during "kubeadm init" to join from the worker node.

In case you lost the token generated by the master node, execute the below command in the master node to make it print again.

    master $ kubeadm token create --print-join-command
    kubeadm join 10.148.0.2:6443 --token wdyfoz.k1wr3dcdc7l9bd2n --discovery-token-ca-cert-hash s
    ha256:d1fa4f7a5d3a5d9b2ba9d32807906fc16fbc16fad5faf81748289faf800add12

Now join the worker node to the cluster.

    worker:~$ sudo kubeadm join 10.160.0.2:6443 --token dg6zhb.naxuxb5wd78tdsqz --discovery-token-ca-cert-hash sha256:af9e87508dc6c2c52b6f51dd88635f2dc9f061dce27368f3ac9913b52522cac3
    [preflight] running pre-flight checks
    I0708 05:23:27.564762 18132 kernel_validator.go:81] Validating kernel version
    I0708 05:23:27.564859 18132 kernel_validator.go:96] Validating kernel config
    [discovery] Trying to connect to API Server "10.160.0.2:6443"
    [discovery] Created cluster-info discovery client, requesting info from "https://10.160.0.2:6443"
    [discovery] Requesting info from "https://10.160.0.2:6443" again to validate TLS against the pinned public key
    [discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "10.160.0.2:6443"
    [discovery] Successfully established connection with API Server "10.160.0.2:6443"
    [kubelet] Downloading configuration for the kubelet from the "kubelet-config-1.11" ConfigMap in the kube-system namespace
    [kubelet] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
    [kubelet] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
    [preflight] Activating the kubelet service
    [tlsbootstrap] Waiting for the kubelet to perform the TLS Bootstrap...
    [patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "instance-2" as an annotation

    This node has joined the cluster:
    * Certificate signing request was sent to master and a response
     was received.
    * The Kubelet was informed of the new secure connection details.

    Run 'kubectl get nodes' on the master to see this node join the cluster.
    worker:~$

Sometime when you run the above command, it may complain that the "ip_vs" kernel module is not loaded.  To solve this issue, you may need to do download and the missing kernel modules.

    worker:~$ sudo apt install ip_vs
    worker:~$ modprobe ip_vs

Also, as mentioned earlier, turn off the Linux swap service before running the above join command, using "*swapoff -a*" command.

## Verify if the cluster nodes are up and running:

Use the "kubectl get nodes" command to check if worker node was able to talk to the master node and join the cluster.  Verify if both master and worker are ready to run pods.

    master:~$ kubectl get nodes
    NAME STATUS ROLES AGE VERSION
    instance-1 Ready master 14m v1.11.0
    instance-2 NotReady  47s v1.11.0
    master:~$ kubectl get nodes
    NAME STATUS ROLES AGE VERSION
    instance-1 Ready master 15m v1.11.0
    instance-2 Ready  1m v1.11.0

That's all folks.  We are done with installing Kubernetes on a bare metal server.

### Spin an NGINX pod in the cluster

We should use the master node to execute any cluster/pod management commands.

    master:~$ kubectl run nginx --image=nginx --port=80
    deployment.apps/nginx created

Verify that the pod is up and running.

    master:~$ kubectl get pods
    NAME READY STATUS RESTARTS AGE
    nginx-6f858d4d45-b7nk5 0/1 ContainerCreating 0 7s
    master:~$ kubectl get pods
    NAME READY STATUS RESTARTS AGE
    nginx-6f858d4d45-b7nk5 1/1 Running 0 31s

To get more details about the pod, use the 'kubectl describe' command.

    master:~$ kubectl describe pods
    Name: nginx-6f858d4d45-b7nk5
    Namespace: default
    Node: instance-2/10.160.0.3
    Start Time: Sun, 08 Jul 2018 05:25:21 +0000
    Labels: pod-template-hash=2941480801
     run=nginx
    Annotations: 

    Status: Running
    IP: 192.168.56.1
    Controlled By: ReplicaSet/nginx-6f858d4d45
    Containers:
     nginx:
     Container ID: docker://90431cb74d4f3e3abf77f3344277ba5679137cfad54a473f56d381d1ad0f3a6d
     Image: nginx
     Image ID: docker-pullable://nginx@sha256:a65beb8c90a08b22a9ff6a219c2f363e16c477b6d610da28fe9cba37c2c3a2ac
     Port: 80/TCP
     Host Port: 0/TCP
     State: Running
     Started: Sun, 08 Jul 2018 05:25:35 +0000
     Ready: True
     Restart Count: 0
     Environment: 
     Mounts:
     /var/run/secrets/kubernetes.io/serviceaccount from default-token-n8l4x (ro)
    Conditions:
     Type Status
     Initialized True 
     Ready True 
     ContainersReady True 
     PodScheduled True 
    Volumes:
     default-token-n8l4x:
     Type: Secret (a volume populated by a Secret)
     SecretName: default-token-n8l4x
     Optional: false
    QoS Class: BestEffort
    Node-Selectors: 
    Tolerations: node.kubernetes.io/not-ready:NoExecute for 300s
     node.kubernetes.io/unreachable:NoExecute for 300s
    Events:
     Type Reason Age From Message
     ---- ------ ---- ---- -------
     Normal Scheduled 38s default-scheduler Successfully assigned default/nginx-6f858d4d45-b7nk5 to instance-2
     Normal Pulling 37s kubelet, instance-2 pulling image "nginx"
     Normal Pulled 25s kubelet, instance-2 Successfully pulled image "nginx"
     Normal Created 24s kubelet, instance-2 Created container
     Normal Started 24s kubelet, instance-2 Started container
