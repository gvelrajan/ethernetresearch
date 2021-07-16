---
title: Kubernetes Dashboard Remote Access
description: ""
author: Ganesh Velrajan
tags: [
    Container, Docker, Kubernetes, Dashboard, Remote Access
]
date: 2020-10-29
categories: [
    GeekZone, Kubernetes
]
images: ["/images/kubernetes/kubernetes.jpg"]
---

# Kubernetes Dashboard Remote Access
Kubernetes is a popular cluster and container management/orchestration platform widely used in pulic and private clouds.  Kubernetes Dashboard is a web-based Kubernetes user interface. You can use Dashboard to deploy containerized applications to a Kubernetes cluster, troubleshoot your containerized application, and manage the cluster resources.

SocketXP TLS VPN solution (a lightweight VPN) provides secure remote access to private Kubernetes Clusters in your private cloud or public cloud.  SocketXP also provides a secure public URL to access your local private applications including Kubernetes Dashboard.

SocketXP agent is available as a docker container in the [SocketXP DockerHub Repository](https://hub.docker.com/r/expresssocket/socketxp). Run the SocketXP Docker container as a standalone container (as explained in the below sections) in your Kubernetes cluster to setup remote access to your Kubernetes Dashboard.

## Deploying the Dashboard UI:
Follow the instructions in the [Kubernetes Open Source Project page](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) on how to deploy and setup the Dashboard UI in your Kubernetes Cluster.

[https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

As explained in the Kubernetes official documentation, you need to run the `kubectl` CLI utility in proxy mode to access your dashboard in a web browser.

## Overall Strategy
Here is the overall strategy to setup remote access to your Kubernetes Dashboard:
- Deploy SocketXP VPN agent Docker container in your K8 cluster.
- Install the `kubectl` CLI utility locally on your laptop.
- Setup the kubectl config file in your laptop with SocketXP Public URL, K8 SSL Certs, and Key.
- Remote access your private Kubernetes cluster from your laptop using the `kubectl` CLI utility.
- Run kubectl in proxy mode in your laptop.
- Access your Kubernetes dashboard in a web browser via the local kubectl proxy.

## SocketXP Agent Docker Container Deployment:
First go to [SocketXP Portal](https://porta.socketxp.com/). Signup for a free account and get your authtoken there.  Use the authtoken to create a Kubernetes secret as shown below.

``` sh
kubectl create secret generic socketxp-credentials --from-literal=authtoken=[your-auth-token-goes-here]
```
Verify that the secret `socketxp-credentials` got created.

``` sh
$ kubectl get secrets
NAME                   TYPE                                  DATA   AGE
default-token-5skb7    kubernetes.io/service-account-token   3      4h
socketxp-credentials   Opaque                                1      4h
$
``` 
We'll use the below `config.json` file to configure the SocketXP agent Docker container.  In this example, we are trying to create a secure public web URL and a TLS VPN tunnel to the Kubernetes API server.

``` json
$ cat config.json
{ 
    "tunnel_enabled": true, 
    "tunnels" : [{ 
        "destination": "https://kubernetes.default", 
        "protocol": "tls", 
        "custom_domain": "", 
        "subdomain": "" 
    }], 
    "relay_enabled": false, 
}

```
Next create a Kubernetes configmap to store the above SocketXP agent configuration file.

``` sh 
kubectl create configmap socketxp-configmap --from-file=/home/test-user/config.json
```
Verify that the `socketxp-configmap` got created.

``` sh
$ kubectl describe configmaps socketxp-configmap
Name:         socketxp-configmap
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
config.json:
----
{ "tunnel_enabled": true, "tunnels" : [{ "destination": "https://kubernetes.default", "protocol": "tls", "custom_domain": "", "subdomain": "" }], "relay_enabled": false }

Events:  <none>
```

Now that we have created the authtoken secret and the configmap needed by the SocketXP agent, it's time to launch the SocketXP Docker container `expresssocket/socketxp:latest` as a Kubernetes Deployment.

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
        image: expresssocket/socketxp:latest
        env:
          - name: AUTHTOKEN
            valueFrom:
              secretKeyRef:
                name: socketxp-credentials
                key: authtoken
        volumeMounts:
        - name: config-volume
          mountPath: /data
      volumes:
        - name: config-volume
          configMap:
            # Provide the name of the ConfigMap containing the files you want
            #to add to the container
            name: socketxp-configmap
``` 
::: warning Note:
We have created a separate volume named `config-volume` and mounted it under `/data` directory inside the container, so that the `socketxp-configmap` will be available as a `config.json` file under the `/data` directory in the running container.
:::

Next, check if the pods are created from the deployment and running.

``` sh
$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
socketxp-75cb4dd7c9-bhxfp   1/1     Running   0          4s
$
```
Now you can retrieve the SocketXP Public URL created for your Kubernetes API server from the SocketXP Portal Page at: [https://portal.socketxp.com/#/tunnels](https://portal.socketxp.com/#/tunnels) or from the pod logs as shown below.

```
$ kubectl logs socketxp-75cb4dd7c9-bhxfp
...
...

Login Succeeded.
User [] Email [test-user@gmail.com].

Connected.
Public URL -> https://test-user-fn4mda420.socketxp.com

``` 
You can now use the above SocketXP Public URL to access the Kubernetes Cluster's API server remotely using a `kubectl` utility.

## Local kubectl installation
Install the `kubectl` CLI utility locally on your laptop to remote access your Kubernetes cluster.  Follow the instructions here to download and install `kubectl` on your laptop:

[https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

After you have installed the kubectl CLI utility, overwrite the `kubectl` config file located at $HOME/.kube/config in your laptop with the one from your cluster's master node($HOME/.kube/config).

Next, update the API server URL in your `kubectl` config file to use the SocketXP Public URL `https://test-user-fn4mda420.socketxp.com`, as shown below.  

``` 
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/test-user/.minikube/ca.crt
    server: https://test-user-fn4mda420.socketxp.com 
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
...
...
```
Please ensure that you also copy the client certificate, CA certificate and private key files or authtoken from your Kubernetes cluster's master node to your laptop in the appropriate folder as specified in the kubectl config file.

Verify that the config works fine, using the following command:
```
kubectl config view
```

Now remote access or remote manage your private Kubernetes Cluster from your laptop by executing any `kubectl` command locally:

``` sh
$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
socketxp-75cb4dd7c9-bhxfp   1/1     Running   0          1h
```
::: warning Note:
When you create more than one replica of the SocketXP agent pod using the deployment, each pod would be assigned a unique SocketXP Public URL.  This is because each SocketXP agent pod running in the Kubernetes Cluster will fetch a new Public URL from the SocketXP Cloud Gateway.
:::

## Run kubectl in proxy mode
To remote access your Kubernetes Dashboard, run the `kubectl` CLI utility in proxy mode in your laptop as shown below:

``` 
$ kubectl proxy  
Starting to serve on 127.0.0.1:8001
```
Let this command continue to run in the foreground.

## Remote access Kubernetes Dashboard:
Now you can remote access your Kubernetes Dashboard from your laptop using the following local URL via the kubectl proxy.  Kubectl will make Dashboard available at: 

[http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/).

Now you can view or manage your k8 resources.

![kubernetes dashboard remote access](../assets/img/kubernetes-dashboard-remote-access/k8-dashboard-remote-access.png)
