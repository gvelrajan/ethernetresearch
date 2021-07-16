---
title: Kubernetes Cluster Remote Access
description: ""
author: Ganesh Velrajan
tags: [
    Container, Docker, Kubernetes, Cluster, Remote Access
]
date: 2020-10-29
categories: [
    GeekZone, Kubernetes
]
images: ["/images/kubernetes/kubernetes.jpg"]
---

Kubernetes is a popular cluster and container management/orchestration platform widely used in pulic and private clouds.  SocketXP TLS VPN solution (a lightweight VPN) provides secure remote access to private Kubernetes Clusters in your private or public cloud.

SocketXP agent is available as a docker container in the [SocketXP DockerHub Repository](https://hub.docker.com/r/expresssocket/socketxp) and can be run in the following modes in a Kubernetes Cluster:

- <strong>Standalone container</strong> - Runs as a separate deployment
- <strong>Sidecar container</strong> - Runs alongside your application container in a pod


## Standalone Container:
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
You can now use the above SocketXP Public URL to access the Kubernetes Cluster's API server remotely using a `kubectl` utility or directly using your custom application.

If you are using a locally installed `kubectl` utility from your laptop to remotely access the Kubernetes, then update the API server URL in the `kubectl` config file located at $HOME/.kube/config to use the SocketXP Public URL `https://test-user-fn4mda420.socketxp.com`

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
Please ensure that you also copy the client certificate, CA certificate and private key files from your Kubernetes cluster's master node to your laptop in the appropriate folder as specified in the kubectl config file.

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

## Sidecar Container:
You have a containerized application and you want to expose the application via a public URL, run SocketXP agent as a sidecar container.

For example, let's say you want to run an `nginx` server and access it remotely using a public URL.  We could run SocketXP agent Docker container as a sidecar container that runs alongside the `nginx` server.

Follow the same steps mentioned in the previous section to create a Kubernetes secret for storing the authtoken and a configmap for storing the SocketXP config file.  

But update the `tunnels[].destination` in the SocketXP config file to the nginx server's local web URL, which is `http://localhost:80`.  Note the presence of `http` (and not `https`)in the local URL.  This is unlike the Kubernetes API server's local web URL shown in the previous section.  

The nginx server used in this example is not configured to use it's own SSL certificate and private key.  It's a HTTP server and not a HTTPS server.

So update the `protocol` field in the config file to `http`.   

However, if you have setup your application as a HTTPS server with its SSL certificate and private key, then set the `protocol` as `tls`.

``` json
$ cat config.json
{ 
    "tunnel_enabled": true, 
    "tunnels" : [{ 
        "destination": "http://localhost:80", 
        "protocol": "http", 
        "custom_domain": "", 
        "subdomain": "" 
    }], 
    "relay_enabled": false, 
}
```
And the `deployment.yaml` file should look this.  

``` yaml
$ cat deployment.yaml 
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
      # nginx container
      - name: nginx
        image: nginx:latest
        ports:
          - containerPort: 80      
      # socketxp container
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
            name: socketxp-configmap
```

The only major difference between this `deployment.yaml` file and the one used in the previous section is the presence of the `nginx` container specifications under the `containers spec` section of the file.

Create a deployment using the `deployment.yaml` file and check the status of the pods.

``` sh
$ kubectl apply -f deployment.yaml 
deployment.apps/socketxp created
``` 

``` bash
$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
socketxp-6477b747d-llrrw   2/2     Running   0          5m
```
Retrieve the SocketXP Public URL assigned to your pod from the portal page or from the pod container's logs as shown below.

``` bash
$ kubectl logs socketxp-6477b747d-llrrw socketxp
...
...

Using config file: /data/config.json
Login Succeeded.
User [] Email [test-user@gmail.com].

Connected.
Public URL -> https://test-user-3bui06m5.socketxp.com
```
Access the above SocketXP public URL in a browser and you should see the welcome message from the `nginx` server as shown below.

![kubernetes cluster docker container remote access](../assets/img/kubernetes-cluster-remote-access/nginx-pod.png)

::: warning Note:
When you create more than one replica of your application pod, each pod would be assigned a unique SocketXP Public URL.  This is because each SocketXP agent running as a sidecar container inside the application pod will fetch a new Public URL from the SocketXP Cloud Gateway.
:::