---
title: Kubernetes - How to Install Minikube in a VM
description: ""
author: Ganesh Velrajan
tags: [
    Container, Docker, Kubernetes, Minikube
]
date: 2018-10-24
categories: [
    GeekZone, Kubernetes
]
images: ["/images/kubernetes/kubernetes.jpg"]
---

## What is Minikube

Minikube is a miniaturized version of the Kubernetes Cluster Platform.

You can install minikube in your laptop, for development, testing and customer demo purposes.

Minikube provides a single node cluster, to run and test application containers.  
It is not a scalable software, like the "full" Kubernetes Cluster Platform .

You can run only very few pods or containers in them.

Minikube is not used in production. It is just a toy for playing with container images on your laptop.

## Find online courses from author Ganesh Velrajan here.

### [Docker and Kubernetes Courses in Udemy at Author Discount Rates.](http://www.ethernetresearch.com/online-courses-training/)

  

###  Minikube Installation Steps

In this Demo, we will show you, how to install minikube on Ubuntu Virtual Machine running in Google Cloud Platform.

Follow the same procedure to install Minikube anywhere, including  installing it in your Lab Server or in your laptop.

This is the script I'll use to download Minikube software.

    gannygans@minikube-cluster:~$ cat minikube-install.sh
    #Install KubeCTL
    sudo apt-get update && sudo apt-get install -y apt-transport-https 

    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

    sudo touch /etc/apt/sources.list.d/kubernetes.list 

    echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list

    sudo apt-get update
    sudo apt-get install -y kubectl

    #Install MiniKube:
    curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.28.2/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

    #Install Docker:
    sudo apt-get update   
   && sudo apt-get install -qy docker.io

Run the installation script.

    gannygans@minikube-cluster:~$ sh minikube-install.sh

Check the version of the minikube installed.

    gannygans@minikube-cluster:~$ minikube version

    minikube version: v0.28.2

The minikube software installation is complete now.  Let’s start the minikube cluster.

Set the "*--vm-driver"* option to NONE, when you install and run minikube inside a virtual machine.

**Note**: If you plan to install minikube directly on your laptop, you should install the oracle virtualbox hypervisor, before you install minikube.  
Also set the "*--vm-driver*" option to virtualbox.

    gannygans@minikube-cluster:~$ sudo minikube start --vm-driver=none 
    There is a newer version of minikube available (v0.29.0). Download it here:
    https://github.com/kubernetes/minikube/releases/tag/v0.29.0
    To disable this notification, run the following:
    minikube config set WantUpdateNotification false
    Starting local Kubernetes v1.10.0 cluster...
    Starting VM...
    Getting VM IP address...
    Moving files into cluster...
    Downloading kubeadm v1.10.0
    Downloading kubelet v1.10.0
    Finished Downloading kubelet v1.10.0
    Finished Downloading kubeadm v1.10.0
    Setting up certs...
    Connecting to cluster...
    Setting up kubeconfig...
    Starting cluster components...
    Kubectl is now configured to use the cluster.
    ===================
    WARNING: IT IS RECOMMENDED NOT TO RUN THE NONE DRIVER ON PERSONAL WORKSTATIONS
            The 'none' driver will run an insecure kubernetes apiserver as root that may leave the host vulnerable to CSRF attacks
    When using the none driver, the kubectl config and credentials generated will be root owned and will appear in the root home directory.
    You will need to move the files to the appropriate location and then set the correct permissions.  An example of this is below:
            sudo mv /root/.kube $HOME/.kube # this will write over any previous configuration
            sudo chown -R $USER $HOME/.kube
            sudo chgrp -R $USER $HOME/.kube
            sudo mv /root/.minikube $HOME/.minikube # this will write over any previous configuration
            sudo chown -R $USER $HOME/.minikube
            sudo chgrp -R $USER $HOME/.minikube
    This can also be done automatically by setting the env var CHANGE_MINIKUBE_NONE_USER=true
    Loading cached images from config file.

Follow the instructions in the output(highlighted in red above)  
Copy paste the commands highlighted in blue color in the command output above, as the instruction says.

    gannygans@minikube-cluster:~$ sudo mv /root/.kube $HOME/.kube # this will write over any previous configuration
    mv: cannot stat '/root/.kube': No such file or directory
    gannygans@minikube-cluster:~$ sudo chown -R $USER $HOME/.kube
    gannygans@minikube-cluster:~$ sudo chgrp -R $USER $HOME/.kube
    gannygans@minikube-cluster:~$ sudo mv /root/.minikube $HOME/.minikube # this will write over any previous configuration
    mv: cannot stat '/root/.minikube': No such file or directory
    gannygans@minikube-cluster:~$ sudo chown -R $USER $HOME/.minikube
    gannygans@minikube-cluster:~$ sudo chgrp -R $USER $HOME/.minikube
    gannygans@minikube-cluster:~$

Now we can run the minikube and kubectl commands as a non-root user.  Let's check the status of the minikube node.

    gannygans@minikube-cluster:~$ kubectl get nodes
    NAME STATUS ROLES AGE VERSION
    minikube Ready master 30m v1.10.0
    gannygans@minikube-cluster:~$

To get more information about the node, use the describe node command.

    gannygans@minikube-cluster:~$ kubectl describe nodes
    Name: minikube
    Roles: master
    Labels: beta.kubernetes.io/arch=amd64
    beta.kubernetes.io/os=linux
    kubernetes.io/hostname=minikube
    node-role.kubernetes.io/master=
    Annotations: node.alpha.kubernetes.io/ttl: 0
    volumes.kubernetes.io/controller-managed-attach-detach: true
    CreationTimestamp: Fri, 28 Sep 2018 07:22:57 +0000
    Taints: <none>
    Unschedulable: false
    Conditions:
    ...

    ...
    KubeletReady kubelet is posting ready status. AppArmor enabled
    Addresses:
    InternalIP: 10.160.0.4
    Hostname: minikube
    Minikube is now ready to run pods.


    ...

    ...

Minikube cluster is now ready to run pods.
