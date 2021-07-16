---
title: Kubernetes Pod Remote SSH Access
description: ""
author: Ganesh Velrajan
tags: [
    Container, Docker, Kubernetes, Pod, Remote Access
]
date: 2020-10-29
categories: [
    GeekZone, Kubernetes
]
images: ["/images/kubernetes/kubernetes.jpg"]
---

Kubernetes is a popular cluster and container management/orchestration platform widely used in public and private clouds.  

SocketXP TLS VPN solution is a lightweight VPN that provides secure remote SSH access to pods running private Kubernetes Clusters in your on-prem cloud or public cloud or multi-cloud or hybrid cloud.

SocketXP Solution provides two options to SSH into your Kubernetes pods:
- Install and run SocketXP agent inside your app container
- Run SocketXP agent container as a standalone container

## Option #1: Install SocketXP inside your app container:

Let's assume for the illustration purpose that you have the following helloworld nodejs app.

``` sh
$cat myapp.js 
var http = require('http');
//create a server object:
http.createServer(function (req, res) {
res.writeHead(200, {'Content-Type': 'text/html'});
res.write('<h1>Hello World!</h1>'); //write a response to the client
res.end(); //end the response
}).listen(80); //the http server object listens on port 80
```
And your Dockerfile looks something like this:

``` sh
$cat Dockerfile 
FROM centos:latest

# your app specific stuff 
RUN yum -y install nodejs
RUN mkdir -p /usr/src/app
COPY ./myapp.js /usr/src/app
WORKDIR /usr/src/app
EXPOSE 80 
ENTRYPOINT ["node","myapp.js"]
```
You might build a Docker container image with the name `helloworld`, as shown below:
``` sh
$ docker build -t socketxp/helloworld .
```
And run your `helloworld` Docker container as shown below:
``` sh
$ docker run --name "helloworld" -p 80:8000 socketxp/helloworld
```
Now you could access your app on port 8000 locally on your host machine as:
``` sh
$ curl http://localhost:8000
<h1>Hello World!</h1>
```
Alternatively, you could run your `helloworld` Docker container as a pod in a Kubernetes cluster using the following `deployment.yaml`.

``` yaml
kube-master-node$ cat deployment.yaml 
apiVersion: apps/v1
kind: Deployment                                          
metadata:
  name: helloworld
spec:
  replicas: 2                                             
  minReadySeconds: 15
  strategy:
    type: RollingUpdate                                   
    rollingUpdate: 
      maxUnavailable: 1                                   
      maxSurge: 1                                         
  template:                                              
    metadata:
      labels:
        app: helloworld                                   
    spec:
      containers:
        - image: socketxp/hello-world:latest 
          imagePullPolicy: Always                         
          name: helloworld 
          ports:
            - containerPort: 80
```
You could create a Kubernetes service using the `service.yaml` shown below to expose your `helloworld` Nodejs app to the internet.

``` yaml
kube-master-node$ cat service.yaml 
apiVersion: v1
kind: Service              
metadata:
  name: helloworld
spec:
  externalIPs:
  - 34.160.0.3
  type: LoadBalancer       
  ports:
  - port: 80               
    protocol: TCP          
    targetPort: 80         
  selector:                
    app: helloworld        
```

``` sh
kube-master-node$ kubectl apply -f service.yaml
```

Now you could access your `helloworld` nodejs app using the public IP as shown below:
``` sh
$curl http://34.160.0.3:80
<h1>Hello World!</h1>
```
So far so good.  

### Add Remote SSH Access Capability
But in the future, if your `helloworld` app behaves erratically and you may want to SSH into one of the pods from a remote machine to debug it, you need a Kubernetes Pod Remote Access Service like SocketXP.

Follow the instructions below to package SocketXP Remote Access Agent binary along with your `helloworld` nodejs app in your `socketxp/helloworld:latest` Docker container, so that you could SSH into your pods from remote.

Here are the instructions to package the SocketXP Remote Access Agent binary into your `helloworld` docker container.  We'll name the new docker container, as `helloworld-ssh` docker container because the new container has remote SSH access capabilities.

First, download SocketXP agent to your local directory
``` sh
curl -O https://portal.socketxp.com/download/linux/socketxp && chmod +wx socketxp
```
Create a shell script file named `script.sh` with the following contents.
``` sh
$cat script.sh 
#!/bin/sh
/var/local/socketxp/socketxp login $AUTHTOKEN --no-auto-update
/var/local/socketxp/socketxp connect tcp://localhost:22 --iot-device-id $HOSTNAME --enable-sshd --ssh-username $SSH_USERNAME --ssh-password $SSH_PASSWORD --no-auto-update &

# Run myapp
node /usr/src/app/myapp.js
```
Here is a sample Dockerfile that packages the SocketXP agent binary, the script.sh file, and instructions to run it, along with the contents of your `helloworld` nodejs app Dockerfile.

```sh
$cat Dockerfile 
FROM centos:latest

# your app specific stuff 
RUN yum -y install nodejs
RUN mkdir -p /usr/src/app
COPY ./myapp.js /usr/src/app
WORKDIR /usr/src/app
EXPOSE 8000 
#ENTRYPOINT ["node","myapp.js"]

# SocketXP Environ Variables
ENV LANG C.UTF-8
ENV AUTHTOKEN ""
ENV SSH_USERNAME ""
ENV SSH_PASSWORD ""
ENV IOT_DEVICE_ID ""

# Install and run SocketXP agent binary
RUN mkdir -p /var/local/socketxp
COPY script.sh /var/local/socketxp/
COPY socketxp /var/local/socketxp/
WORKDIR /var/local/socketxp
RUN chmod a+x socketxp
EXPOSE 22

ENTRYPOINT ["/bin/sh", "script.sh"]
```
Now let's create a new Docker image named `helloworld-ssh` as shown below:

``` sh
$ docker build -t socketxp/helloworld-ssh .
```

### Run as Docker Container on Host Machine:
Now you could run this Docker container on your host machine as shown below:
``` sh
$ docker run --name helloworld-ssh -d -p 80:8000 -e SSH_USERNAME="test-user" -e SSH_PASSWORD="pa$$word123" -e AUTHTOKEN="<your-socketxp-authtoken-goes-here>" socketxp/helloworld-ssh:latest
```
You could access your nodejs app from your host machine as shown below:
``` sh
$curl http://localhost:8000
<h1>Hello World!</h1>
```
You could SSH into your Docker container from the [SocketXP Portal's IoT Device Page](https://portal.socketxp.com/#/devices).  Click the terminal icon next to your container listed in the page.  [**Note**: The Docker container ID of the app will be listed under the DeviceID column].  It will take you to an SSH login window in a new tab.  Provide the same SSH login credentials that you provided in the "docker run" command above.  You'll get shell access to your `helloworld-ssh` Docker container.

### Run as Kubernetes Pod:
Alternatively, you could run the `socketxp/helloworld-ssh:latest` Docker container as a Kubernetes pod/deployment/service and you could remote SSH into this pod using the SSH credentials you have setup.

Here is how your new deployment.yaml file would look like:

``` yaml
$cat deployment.yaml 
apiVersion: apps/v1
kind: Deployment                                          
metadata:
  name: helloworld-ssh
spec:
  replicas: 2                                             
  minReadySeconds: 15
  strategy:
    type: RollingUpdate                                   
    rollingUpdate: 
      maxUnavailable: 1                                   
      maxSurge: 1                                         
  template:                                              
    metadata:
      labels:
        app: helloworld-ssh    
    spec:
      containers:
      - name: helloworld-ssh
        image: socketxp/helloworld-ssh:latest
        imagePullPolicy: Always
        env:
          - name: AUTHTOKEN
            valueFrom:
              secretKeyRef:
                name: socketxp-credentials
                key: authtoken 
          - name: SSH_USERNAME 
            valueFrom:
              secretKeyRef:
                name: socketxp-credentials
                key: ssh_username 
          - name: SSH_PASSWORD
            valueFrom:
              secretKeyRef:
                name: socketxp-credentials
                key: ssh_password 
```
But, before you could apply this `deployment.yaml` in a Kubernetes cluster, you need to create the environment variables used by SocketXP agent in the `deployment.yaml` file above.

### SocketXP ENV variables creation
First go to [SocketXP Portal](https://portal.socketxp.com). Signup for a free account and get your authtoken there. Use the authtoken to create a Kubernetes secret as shown below.
``` sh
kubectl create secret generic socketxp-credentials --from-literal=authtoken="<your-auth-token-goes-here>" --from-literal=ssh_username="test-user" --from-literal=ssh_password="pa$$word123"
```
You could specify your own SSH login username and password in the above CLI. You'll be prompted to provide the same username and password when you attempt to SSH login using any SSH client.

Verify that the secret `socketxp-credentials` got created.

``` sh
$ kubectl get secrets
NAME                   TYPE                                  DATA   AGE
default-token-5skb7    kubernetes.io/service-account-token   3      4h
socketxp-credentials   Opaque                                1      4h
$
```

Now that we have created the secrets needed by the SocketXP agent, it's time to launch the `helloworld-ssh` deployment using the `deployment.yaml` file above.

``` sh
kube-master-node$ kubectl apply -f deployment.yaml
```
Your `helloworld` loadbalancer service could continue to run as it is without any changes.  This is because SocketXP Remote Access Agent doesn't require any Kubernetes Service to be created for it.  The agent would automatically create a secure TLS VPN tunnel to the SocketXP Cloud Gateway, from where you could SSH into your pods.

Go to the SocketXP Cloud Gateway Portal's [IoT Devices Page](https://portal.socketxp.com/#/devices) using your browser.  You'll see your pods (2 replicas) listed as IoT Devices.  You could SSH into your pods by clicking the terminal icon next to them.  It will open up an SSH Login window in a new tab.  Enter the same SSH login credentials you provided when creating the Kubernetes secret `socketxp-credentials`.  

After successful authentication, you'll be provided with the pod's shell prompt.

## Option #2: Run SocketXP as a Standalone Container:
In this second option, you need to run SocketXP Agent Docker container as a standalone container pod in your Kubernetes cluster, so that you could get remote SSH access to other pods(your apps) running in the cluster, through this SocketXP Agent pod.

SocketXP Agent Docker container is shipped with an SSH server daemon and Kubernetes [`kubectl`](https://kubernetes.io/docs/tasks/tools/install-kubectl/) CLI utility packaged in it.  SocketXP Docker container can be downloaded from the [SocketXP DockerHub Repository](https://hub.docker.com/r/expresssocket/socketxp-kubectl).  This turn-key solution from SocketXP lets you to setup a remote SSH into a pod in your private cluster, in just few minutes. 

Let's get started.

First go to [SocketXP Portal](https://porta.socketxp.com/). Signup for a free account and get your authtoken there.  Use the authtoken to create a Kubernetes secret as shown below.

``` sh
kubectl create secret generic socketxp-credentials --from-literal=authtoken=[your-auth-token-goes-here] --from-literal=ssh_username="test-user" --from-literal=ssh_password="password123"
```
You could specify your own SSH login username and password in the above CLI.  You'll be prompted to provide the same username and password when you attempt to SSH login using any SSH client.

Verify that the secret `socketxp-credentials` got created.

``` sh
$ kubectl get secrets
NAME                   TYPE                                  DATA   AGE
default-token-5skb7    kubernetes.io/service-account-token   3      4h
socketxp-credentials   Opaque                                1      4h
$
``` 

Now that we have created the secrets needed by the SocketXP agent, it's time to launch the SocketXP Docker container `expresssocket/socketxp-kubectl:latest` as a Kubernetes Deployment.

Here is the `deployment.yaml` file we'll use to create a standalone SocketXP agent deployment.

``` yaml
$cat deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: socketxp
  labels:
    app: socketxp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: socketxp
  template:
    metadata:
      labels:
        app: socketxp
    spec:
      containers:
      - name: socketxp
        image: expresssocket/socketxp-kubectl:latest
        imagePullPolicy: Always
        env:
          - name: AUTHTOKEN
            valueFrom:
              secretKeyRef:
                name: socketxp-credentials
                key: authtoken 
          - name: SSH_USERNAME 
            valueFrom:
              secretKeyRef:
                name: socketxp-credentials
                key: ssh_username 
          - name: SSH_PASSWORD
            valueFrom:
              secretKeyRef:
                name: socketxp-credentials
                key: ssh_password 
``` 
Now create a deployment using the `deployment.yaml` file and check the status of the pods.

``` sh
$ kubectl apply -f deployment.yaml 
deployment.apps/socketxp created
```
``` sh
$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
socketxp-75cb4dd7c9-bhxfp   1/1     Running   0          4s
$
```
Check the pod logs to see if everything went on well as we had expected and there are no errors.

```
$ kubectl logs socketxp-75cb4dd7c9-bhxfp
...
...

Login Succeeded.
User [] Email [test-user@gmail.com].

Connected.
TCP tunnel [test-user-vbmyrruv] created.
Access the tunnel using SocketXP agent in IoT Slave Mode.

``` 
You can now SSH into this pod from the SocketXP Portal's IoT Device Access Page: [https://portal.socketxp.com/#/devices]("https://portal.socketxp.com/#/devices").  Click the terminal icon there, to SSH into the `socketxp-75cb4dd7c9-bhxfp` pod.

You'll be prompted to enter the SSH username and the SSH password.  This should match the ones you provided earlier when you configured the kubernetes secret `socketxp-credentials` above.

Once you are logged in, you'll be provided with the pod container's root shell prompt, which would look like this.
``` sh
[root@socketxp-75cb4dd7c9-bhxfp socketxp]#
```
The `expresssocket/socketxp-kubectl` container is packaged with the `kubectl` utility in it.  So you could execute any kubectl command from this pod shell.
```
[root@socketxp-75cb4dd7c9-bhxfp socketxp]# kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
socketxp-75cb4dd7c9-bhxfp   1/1     Running   0          7m
[root@socketxp-75cb4dd7c9-bhxfp socketxp]# 
```
You could now connect to the shell of any other pod in the cluster, using the following kubectl command:
``` sh
kubectl exec --stdin --tty <pod-name> -c <container-name>
```
You can skip the `-c` option in the above command if the pod has only one container.

### Security Best Practices

Enterprises usually run Kubernetes clusters privately and not publicly.  They don't expose the clusters to the internet for security reasons. Ideally Kubernetes clusters should not be directly exposed to the internet by configuring the ClusterIP or the NodeIP with a public IP address.  Only the application that needs to be exposed to the internet, should be exposed to the internet via a load-balancer configured with public IP.  This approach reduces the attack surface for any potential attacks.

Moreover, you should use a VPN solution to gain remote SSH access to the private cluster and its resources.

But most VPN solutions in the market are heavy, clunky and cost more money.  It is highly recommended to use a lightweight VPN soloution such as SocketXP to securely access your private Kubernetes cluster resources.