---
title: Autoupdate Kubernetes Deployment on GitHub Webhook
description: ""
author: Ganesh Velrajan
tags: [
    Container, Docker, Kubernetes, CI/CD, GitHub, Webhook
]
date: 2020-07-25
categories: [
    GeekZone, Kubernetes
]
images: ["/images/kubernetes/kubernetes.jpg"]
---

In this article, I'll discuss how to autoupdate a Kubernetes workload or deployment on receiving a GitHub Webhook when a new version of the app is released.

## Prerequisites

For understanding and following the instructions in this demo you need to have a basic understanding and some working knowledge of the following tools.

-   GitHub
-   Docker
-   Kubernetes Cluster
-   Helm
-   SocketXP

## Continuous Deployment Strategy

Our goal is to upgrade our demo app in a GitOps fashion, meaning we don't want to upgrade whenever a new checkin happens into our app git repo but when a change or release is created in our Devops template or script git repo.

In this demo, I'll show you how to create a separate git repo for the helm chart of our app. Whenever a new release is tagged on the master branch for this helm chart git repo we'll upgrade the helm chart release in our Kubernetes Cluster, which in turn will upgrade our http-server app in our Kubernetes cluster.

![Kubernetes-autoupdate-github-webhook-helm-chart](https://www.socketxp.com/wp-content/uploads/2020/07/Kubernetes-autoupdate-github-webhook-helm-chart.png){.aligncenter .wp-image-1198 .size-full width="1024" height="768"}

## Demo App

We'll use the same demo app we used in our last week's demo. I'm displaying it again over here for convenience.

``` {.source-code}
$cat http-server.js

var http = require('http');

var port = 8080

var handleRequest = function(request, response) {

console.log('Received HTTP request for URL: ' + request.url);

response.writeHead(200);

response.end('Hello World!, v1.0');

};

var server = http.createServer(handleRequest);

server.listen(port);
```

Also the Dockerfile used to build our http-server app from our last week's demo.

``` {.source-code}
FROM alpine:latest

RUN apk update && apk add nodejs

RUN mkdir -p /usr/src/app

COPY ./http-server.js /usr/src/app

WORKDIR /usr/src/app

EXPOSE 8080

CMD ["node","http-server.js"]
```

Refer to our [**last week's demo article**](https://www.socketxp.com/webhookrelay/docker-compose-auto-update-github-webhook/) on how to build a version 1.0 of the docker image and publish it to a DockerHub registry.

## Kubernetes Cluster

Install Kubernetes Cluster or a Minikube Cluster following the instructions here:

-   [How to install Kubernetes on Ubuntu](http://www.ethernetresearch.com/geekzone/kubernetes-tutorial-how-to-install-kubernetes-on-ubuntu/)
-   [How to install minikube in a VM](http://www.ethernetresearch.com/kubernetes/kubernetes-how-to-install-minikube-in-a-vm/)

Check the nodes and the pods available.

``` {.source-code}
$kubectl get nodes

NAME       STATUS   ROLES    AGE     VERSION

minikube   Ready    master   3h45m   v1.18.3
```

``` {.source-code}
$kubectl get pods

No resources found in default namespace.

$
```

## Helm

I have already created an helm chart for this demo and is available at: **<https://github.com/socketxp-com/kubernetes-webhook-autoupdate-helm-chart>**

But if you want to create your own helm chart follow the instructions below to install helm and create a helm chart.

``` {.source-code}
$ brew install helm
```

``` {.source-code}
$ helm create http-server-chart

$ ls

http-server-chart

$ cd http-server-chart

$ ls

Chart.yaml charts templates values.yaml

$
```

Now edit the values.yaml file and update the repository and tag fields to your app's docker image name and tag. Also update templates such as deployment.yaml or service.yaml under the templates folder, if required.

Commit the changes to your chart into a git repository, so that the changes/versions can be tracked. Also this will help in upgrading the corresponding workloads in a Kubernetes cluster, in a GitOps fashion.

## Run SocketXP on the master node

Download and install SocketXP agent on the master node of your Kubernetes cluster's master node. You can follow the [**SocketXP download instructions here**](https://www.socketxp.com/download/).

Next jump into the root directory of the git cloned chart repo and execute the below command. The repo already has the files "update-kube.sh" and "filter-rules.sh" from the last week's demo. The only change is in the update-kube.sh file to execute a "helm upgrade" command instead of "docker-compose up -d" command.

``` {.source-code}
$ cat update-kube.sh

#!/bin/bash



git pull

helm upgrade http-server http-server-chart
```

Execute the SocketXP relay command.

``` {.source-code}
$ socketxp relay http://localhost:8443 --exec ./update-kube.sh --filter filter-rules.json



connected.

Public URL -> https://webhook.socketxp.com/ganeshvelrajan-rmzlayq9
```

Now pick up this SocketXP Public Webhook URL and go to your GitHub Webhook Settings page and paste this URL in the Public Webhook URL text box. Make sure you set the content type of GitHub webhook to "application/json". Also set the secret with your own secret. This secret will be used to validate the sender when a GitHub webhook is received at your Kubernetes Cluster master node.

![Kubernetes Github Webhook-socketxp](https://www.socketxp.com/wp-content/uploads/2020/07/Kubernetes-Github-Webhook-.png){.aligncenter .wp-image-1201 .size-full width="2484" height="1802"}

## Install Helm Chart

Install the helm chart manually using the following command and bring up the http-server app in the kubernetes cluster. We need to do this only once when we bring up the app for the very first time. Thereafter, we'll automate the upgrade process using SocketXP.

``` {.source-code}
$ls

LICENSE filter-rules.json update-kube.sh

README.md http-server-chart

$helm install http-server http-server-chart/

NAME: http-server

LAST DEPLOYED: Fri Jul 10 11:59:39 2020

NAMESPACE: default

STATUS: deployed

REVISION: 1

NOTES:

1. Get the application URL by running these commands:

export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services http-server-http-server-chart)

export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")

echo http://$NODE_IP:$NODE_PORT
```

Let's follow the instructions in the above output and copy paste the above commands.

``` {.source-code}
$export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services http-server-http-server-chart)

export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")

$  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")

$echo http://$NODE_IP:$NODE_PORT

http://192.168.99.100:32194

$
```

Let's try to access our http-server application at the above mentioned URL.

``` {.source-code}
$curl http://192.168.99.100:32194

Hello World!, v1.0
```

Great. Our app is running version 1.0 as expected.

Next verify the helm release version currently deployed.

``` {.source-code}
$helm ls

NAME        NAMESPACE REVISION UPDATED                              STATUS  CHART                  APP VERSION

http-server default  1        2020-07-10 11:59:39.651311 +0530 IST deployed http-server-chart-0.1.0 1.16.0
```

## Begin Demo - Upgrade the app and the chart

Now let's edit our http-server app like we did in the last week's demo and push a docker container image version 2.0 to DockerHub. Please refer to our last week's demo article if you forgot the instruction for doing so.

Next let's edit the helm-chart and upgrade the image version and the chart version to 2.0 and 0.2.0 respectively.

``` {.source-code}
$vim http-server-chart/values.yaml

# Default values for http-server-chart.

# This is a YAML-formatted file.

# Declare variables to be passed into your templates.



replicaCount: 1



image:

repository: gvelrajan/http-server

pullPolicy: IfNotPresent

# Overrides the image tag whose default is the chart appVersion.

tag: "v2.0"

...

...
```

``` {.source-code}
...

# This is the chart version. This version number should be incremented each time you make changes

# to the chart and its templates, including the app version.

# Versions are expected to follow Semantic Versioning (https://semver.org/)

version: 0.2.0

...

...
```

Commit the changes and push to GitHub.

``` {.source-code}
$git add .

$git commit -m "image version and chart version updated"

[master 1765b2e] image version and chart version updated

4 files changed, 35 insertions(+), 2 deletions(-)

create mode 100644 filter-rules.json

create mode 100755 update-kube.sh

$git push

Counting objects: 7, done.

Delta compression using up to 8 threads.

Compressing objects: 100% (7/7), done.

Writing objects: 100% (7/7), 839 bytes | 839.00 KiB/s, done.

Total 7 (delta 3), reused 0 (delta 0)

remote: Resolving deltas: 100% (3/3), completed with 3 local objects.

To https://github.com/socketxp-com/kubernetes-webhook-autoupdate-helm-chart.git

2f12153..1765b2e  master -> master
```

As soon as we do git push for the chart into its online repo, GitHub will trigger a webhook notification for the git push event. SocketXP will receive the webhook, validate the signature and try to match it with the fiter rules. Since this is not a release webhook, SocketXP will throw a mismatch error and not further process this webhook.

``` {.source-code}
Connected.

Public URL -> https://webhook.socketxp.com/ganeshvelrajan-rmzlayq9



Webhook received:

POST / HTTP/1.1

Host: webhook.socketxp.com

Accept: */*

Accept-Encoding: gzip

Content-Length: 9397

Content-Type: application/json

User-Agent: GitHub-Hookshot/f2f2346

X-Github-Delivery: 4aba8354-c279-11ea-980e-6cd6cfda12a3

X-Github-Event: push

X-Hub-Signature: sha1=14e1f0a3d368d9b5dadd7d41cd575af2f45065ce



{"ref":"refs/heads/master","before":"2f12153ef944b939bc81712e1d047d281d2deabe","after":"1765b2ecd58ad3c6d3fec15c7b3198063b543405","repository":{"id":278398971,"node_id":"MDEwOlJlcG9zaXRvcnkyNzgzOTg5NzE=","name":"kubernetes-webhook-autoupdate-helm-chart",

...

]}}



#Rule: Webhook signature matched.

#Rule: ref in payload didn't match

Webhook received didn't match with the filter rules.

Command not executed.
```

## Release version 0.2.0 of the chart

So this proves that our http-server app will not be upgraded on receiving just any ad-hoc wehbook notifications from GitHub. It gets upgraded only on receiving an official release webhook for the app's helm chart.

Now, let's go ahead and create an official release version 0.2.0 of the helm chart for our app. We do this in the GitHub release page for our app's helm-chart repo.

\[ Note: But before doing this make sure you update the print string in our http-server app to print "v2.0".  Also build a new Docker container image for the upgraded app and tag it as v2.0.   Refer to our last week'd demo if you need to know how to perform these actions\]

![Kubernetes Continuous Deployment Helm Chart GitHub Webhook](https://www.socketxp.com/wp-content/uploads/2020/07/Helm-Chart-Release-Version-0.2.0.png){.aligncenter .wp-image-1202 .size-full width="2106" height="1184"}

SocketXP will receive the release webhook from GitHub. It will process the webhook, validate the signature and match the ref field in the webhook payload with the filter rules.

``` {.source-code}
Webhook received:

POST / HTTP/1.1

Host: webhook.socketxp.com

Accept: */*

Accept-Encoding: gzip

Content-Length: 8725

Content-Type: application/json

User-Agent: GitHub-Hookshot/f2f2346

X-Github-Delivery: b08b5ec8-c27a-11ea-8774-72206af92c06

X-Github-Event: push

X-Hub-Signature: sha1=d6fefb9919e8b322226f6dcd485bd2f2591e80c8



{"ref":"refs/tags/v0.2.0","before":"0000000000000000000000000000000000000000","after":"1765b2ecd58ad3c6d3fec15c7b3198063b543405","repository":{"id":278398971,"node_id":"MDEwOlJlcG9zaXRvcnkyNzgzOTg5NzE=","name":"kubernetes-webhook-autoupdate-helm-chart","full_name":"socketxp-com/kubernetes-webhook-autoupdate-helm-chart",

...

...



]}}



#Rule: Webhook signature matched.

#Rule: ref in payload matched.

All rules matched.



Executing command:  [./update-kube.sh]

From https://github.com/socketxp-com/kubernetes-webhook-autoupdate-helm-chart

* [new tag]         v0.2.0     -> v0.2.0

Already up to date.

Current branch master is up to date.

Release "http-server" has been upgraded. Happy Helming!

NAME: http-server

LAST DEPLOYED: Fri Jul 10 12:42:09 2020

NAMESPACE: default

STATUS: deployed

REVISION: 2

NOTES:

1. Get the application URL by running these commands:

export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services http-server-http-server-chart)

export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")

echo http://$NODE_IP:$NODE_PORT
```

Let's follow the instructions above output and copy paste the above commands in a separate terminal.

``` {.source-code}
$  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services http-server-http-server-chart)

$  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")

$  echo http://$NODE_IP:$NODE_PORT

http://192.168.99.100:32194
```

Now, let's access the URL using curl.

``` {.source-code}
$curl http://192.168.99.100:32194

Hello World!, v2.0
```

Also confirm the kubernetes workload(pod) has the correct versions.

``` {.source-code}
$kubectl describe  pods

Name:         http-server-http-server-chart-5947946f5f-vb5cn

Namespace:    default

Priority:     0

Node:         minikube/192.168.99.100

Start Time:   Fri, 10 Jul 2020 02:30:00 +0530

Labels:       app.kubernetes.io/instance=http-server

app.kubernetes.io/name=http-server-chart

pod-template-hash=5947946f5f

Annotations:

Status:       Running

IP:           172.17.0.5

IPs:

IP:           172.17.0.5

Controlled By:  ReplicaSet/http-server-http-server-chart-5947946f5f

Containers:

http-server-chart:

Container ID:   docker://7deb29b8786bc474374be82b27c33858cad8ec4b86754c29ad6467bf41f4ecc9

Image:          gvelrajan/http-server:v2.0

Image ID:       docker-pullable://gvelrajan/http-server@sha256:2a9ee82f76758758c77659260091380b20e3b3d095efdc6480d66e47c5842476

Port:           8080/TCP

Host Port:      0/TCP

State:          Running

Started:      Fri, 10 Jul 2020 02:30:03 +0530

Ready:          True

Restart Count:  0

Liveness:       http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3

Readiness:      http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3

Environment:

Mounts:

/var/run/secrets/kubernetes.io/serviceaccount from http-server-http-server-chart-token-rmppc (ro)

Conditions:

Type              Status

Initialized       True

Ready             True

ContainersReady   True

PodScheduled      True

Volumes:

http-server-http-server-chart-token-rmppc:

Type:        Secret (a volume populated by a Secret)

SecretName:  http-server-http-server-chart-token-rmppc

Optional:    false

QoS Class:       BestEffort

Node-Selectors:

Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s

node.kubernetes.io/unreachable:NoExecute for 300s

Events:

Type    Reason     Age   From               Message

----    ------     ----  ----               -------

Normal  Scheduled  10h   default-scheduler  Successfully assigned default/http-server-http-server-chart-5947946f5f-vb5cn to minikube

Normal  Pulled     10h   kubelet, minikube  Container image "gvelrajan/http-server:v2.0" already present on machine

Normal  Created    10h   kubelet, minikube  Created container http-server-chart

Normal  Started    10h   kubelet, minikube  Started container http-server-chart

$


```

Also let's check the helm release version currently deployed.

``` {.source-code}
$helm ls

NAME        NAMESPACE REVISION UPDATED                              STATUS  CHART                  APP VERSION

http-server default  2        2020-07-10 12:42:09.288038 +0530 IST deployed http-server-chart-0.2.0 1.16.0

$
```

Everything confirms that we are have successfully upgraded our app to version 2.0 by upgrading the helm chart release version to 0.2.0

## Rolling back to the previous working version

Suppose, the upgrade to version 2.0 of the app introduced some severe bugs or problems that is unacceptable by any user, we would want to quickly go back to the previous working version, so that customers are not affected by our upgrade.

Helm makes the process of rollback very simple. Let's do it. We'll rollback to revision \#1 of the helm chart from revision \#2.

``` {.source-code}
$helm rollback http-server 1

Rollback was a success! Happy Helming!

$helm ls

NAME        NAMESPACE REVISION UPDATED                              STATUS  CHART                  APP VERSION

http-server default  3        2020-07-10 12:53:04.755063 +0530 IST deployed http-server-chart-0.1.0 1.16.0
```

Curl the app's web URL and verify if the rollback is successful.

``` {.source-code}
$curl http://192.168.99.100:32194

Hello World!, v1.0
```

Verify the pods running in the Kubernetes cluster have the right version. Notice that the pod running the old version of our http-server app is terminating and the pod running the new version is up and running.

``` {.source-code}
$kubectl describe  pods

Name:                      http-server-http-server-chart-5947946f5f-vb5cn

Namespace:                 default

Priority:                  0

Node:                      minikube/192.168.99.100

Start Time:                Fri, 10 Jul 2020 02:30:00 +0530

Labels:                    app.kubernetes.io/instance=http-server

app.kubernetes.io/name=http-server-chart

pod-template-hash=5947946f5f

Annotations:

Status:                    Terminating (lasts 10h)

Termination Grace Period:  30s

IP:                        172.17.0.5

IPs:

IP:           172.17.0.5

Controlled By:  ReplicaSet/http-server-http-server-chart-5947946f5f

Containers:

http-server-chart:

Container ID:   docker://7deb29b8786bc474374be82b27c33858cad8ec4b86754c29ad6467bf41f4ecc9

Image:          gvelrajan/http-server:v2.0

Image ID:       docker-pullable://gvelrajan/http-server@sha256:2a9ee82f76758758c77659260091380b20e3b3d095efdc6480d66e47c5842476

Port:           8080/TCP

Host Port:      0/TCP

State:          Running

Started:      Fri, 10 Jul 2020 02:30:03 +0530

Ready:          True

Restart Count:  0

Liveness:       http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3

Readiness:      http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3

Environment:

Mounts:

/var/run/secrets/kubernetes.io/serviceaccount from http-server-http-server-chart-token-rmppc (ro)

Conditions:

Type              Status

Initialized       True

Ready             True

ContainersReady   True

PodScheduled      True

Volumes:

http-server-http-server-chart-token-rmppc:

Type:        Secret (a volume populated by a Secret)

SecretName:  http-server-http-server-chart-token-rmppc

Optional:    false

QoS Class:       BestEffort

Node-Selectors:

Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s

node.kubernetes.io/unreachable:NoExecute for 300s

Events:

Type    Reason     Age   From               Message

----    ------     ----  ----               -------

Normal  Scheduled  10h   default-scheduler  Successfully assigned default/http-server-http-server-chart-5947946f5f-vb5cn to minikube

Normal  Pulled     10h   kubelet, minikube  Container image "gvelrajan/http-server:v2.0" already present on machine

Normal  Created    10h   kubelet, minikube  Created container http-server-chart

Normal  Started    10h   kubelet, minikube  Started container http-server-chart

Normal  Killing    10h   kubelet, minikube  Stopping container http-server-chart





Name:         http-server-http-server-chart-bdd75ff4-7jmbx

Namespace:    default

Priority:     0

Node:         minikube/192.168.99.100

Start Time:   Fri, 10 Jul 2020 02:40:56 +0530

Labels:       app.kubernetes.io/instance=http-server

app.kubernetes.io/name=http-server-chart

pod-template-hash=bdd75ff4

Annotations:

Status:       Running

IP:           172.17.0.4

IPs:

IP:           172.17.0.4

Controlled By:  ReplicaSet/http-server-http-server-chart-bdd75ff4

Containers:

http-server-chart:

Container ID:   docker://239aadd09eb54185b3aa559d5ba3a89f2cb99a30c5b340270b834f9e25de24cb

Image:          gvelrajan/http-server:v1.0

Image ID:       docker-pullable://gvelrajan/http-server@sha256:07e70e778e1002519064503ece0d334291f4fbb7a7196b894f605ff5e3076e18

Port:           8080/TCP

Host Port:      0/TCP

State:          Running

Started:      Fri, 10 Jul 2020 02:40:57 +0530

Ready:          True

Restart Count:  0

Liveness:       http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3

Readiness:      http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3

Environment:

Mounts:

/var/run/secrets/kubernetes.io/serviceaccount from http-server-http-server-chart-token-rmppc (ro)

Conditions:

Type              Status

Initialized       True

Ready             True

ContainersReady   True

PodScheduled      True

Volumes:

http-server-http-server-chart-token-rmppc:

Type:        Secret (a volume populated by a Secret)

SecretName:  http-server-http-server-chart-token-rmppc

Optional:    false

QoS Class:       BestEffort

Node-Selectors:

Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s

node.kubernetes.io/unreachable:NoExecute for 300s

Events:

Type    Reason     Age   From               Message

----    ------     ----  ----               -------

Normal  Scheduled  10h   default-scheduler  Successfully assigned default/http-server-http-server-chart-bdd75ff4-7jmbx to minikube

Normal  Pulled     10h   kubelet, minikube  Container image "gvelrajan/http-server:v1.0" already present on machine

Normal  Created    10h   kubelet, minikube  Created container http-server-chart

Normal  Started    10h   kubelet, minikube  Started container http-server-chart

$
```

That's all folks!! I hope you were able to follow the instructions and auto deploy your app in Kubernetes using SocketXP and Helm chart in a GitOps fashion.

## Conclusion

SocketXP and Helm are two great tools to setup a full-automated Continuous Deployment(CD) pipeline for your app running in your Kubernetes Cluster. Though you might want to expose your app ( a front-end microservice) running in your Kubernetes Cluster to the internet, you wouldn't want to and shouldn't expose your Kubernetes Cluster itself to the internet.

When you are not exposing the Kubernetes Cluster to the internet, it may be challenging to receive webhook notifications from GitHub, Gitlab, DockerHub, GCR and other online version control and artifact repositories. SocketXP solves this problem by quickly creating a secure webhook relay tunnel between the online registry and your Kubernetes Cluster, so that webhooks could reach your private Kubernetes Cluster.

SocketXP is a freemium online webhook relay service, which has free and paid plans for developers, businesses and enterprise customers. [**Sign up for your free SocketXP account here**](https://portal.socketxp.com).
