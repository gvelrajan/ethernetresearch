---
title: Docker Compose Auto Update on GitHub Webhook
description: ""
author: Ganesh Velrajan
tags: [
    GeekZone, Docker, Kubernetes, CI/CD, Docker Compose, GitHub
]
date: 2020-07-23
categories: [
    GeekZone, Docker
]
images: ["/images/docker/docker.png"]
---

In the last week's demo, we discussed how to setup a [**Continuous Deployment(CD) pipeline for a native NodeJS application**](https://www.socketxp.com/webhookrelay/automatically-deploy-github-local-server-git-push/).

In this article, I'll show you how to build a GitOps style CD pipeline that keeps a Docker Compose deployment in sync with a docker-compose.yaml hosted on a git repository.

## What's new in this demo?

We have added some cool new features to SocketXP agent, such as:

-   ability to filter incoming webhooks on user specified filter rules
-   ability to execute a command or a script file when an incoming webhook matches with the user specified rules

Secondly, and more importantly, we'll be running our NodeJS web app inside a Docker container and bring it up using a Docker Compose YAML file.

## Prerequisites

Here are the prerequisites for this demo.

-   GitHub
-   SocketXP Account
-   SocketXP Agent Installation
-   Docker Engine, Docker Compose and DockerHub Account

![Docker-compose-auto-update-github-webhook](https://www.socketxp.com/wp-content/uploads/2020/07/Docker-compose-auto-update-github-webhook.png){.aligncenter .wp-image-1148 .size-full width="1024" height="768"}

Repository with scripts that I used for this article can be found here: <https://github.com/socketxp-com/docker-compose-autoupdate-on-github-webhook>.

## Demo App

We'll use the same nodejs http server that we used in the last week's demo.

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

## Dockerfile

Here is the Dockerfile we'll use to package the above nodejs app into a Docker container.

``` {.source-code}
FROM alpine:latest
RUN apk update && apk add nodejs
RUN mkdir -p /usr/src/app
COPY ./http-server.js /usr/src/app
WORKDIR /usr/src/app
EXPOSE 8080 
CMD ["node","http-server.js"]
```

Let's build the Docker image using the following Docker command.

``` {.source-code}
$docker build -t gvelrajan/http-server:v1.0 .
Sending build context to Docker daemon  56.83kB
Step 1/7 : FROM alpine:latest
 ---> f70734b6a266
Step 2/7 : RUN apk update && apk add nodejs
 ---> Using cache
...
Successfully built b6b06a819b63
Successfully tagged gvelrajan/http-server:v1.0
$
```

Next, let's push this Docker image to DockerHub image registry, so that it can be downloaded by the production team to deploy it.

``` {.source-code}
$docker push gvelrajan/http-server:v1.0
The push refers to repository [docker.io/gvelrajan/http-server]
3b8d0f6b058e: Layer already exists 
947da554eff8: Layer already exists 
cc403a71dcfb: Pushed 
3e207b409db3: Layer already exists 
v1.0: digest: sha256:07e70e778e1002519064503ece0d334291f4fbb7a7196b894f605ff5e3076e18 size: 1154
```

## Docker Compose

Docker Compose is a very helpful tool to bring up various services of an application as Docker containers using a simple single command. In our demo example we have just a single service - the http web service, packaged as a Docker container.

Here is the docker-compose yaml file we'll use to bring up our http-server docker container in production.

``` {.source-code}
$cat docker-compose.yml 
version: '3'
services:
  web:
    image: "gvelrajan/http-server:v1.0"
    ports:
      - "8080:8080"
$
```

Now, bring up the app container by executing the following command.

``` {.source-code}
$docker-compose up -d
Creating network "docker-compose-autoupdate-on-github-webhook_default" with the default driver
Creating docker-compose-autoupdate-on-github-webhook_web_1_fd2bb2cedec9 ... done
```

Check if the docker container is up and running, and listening on the correct port(8080).

``` {.source-code}
$docker ps
CONTAINER ID        IMAGE                        COMMAND                 CREATED             STATUS              PORTS                    NAMES
899ab99abedf        gvelrajan/http-server:v1.0   "node http-server.js"   2 seconds ago       Up 1 second         0.0.0.0:8080->8080/tcp   docker-compose-autoupdate-on-github-webhook_web_1_64dc034e55ac
$
```

## Curl the web server

Verify that our web app is running and serving content on port 8080.

``` {.source-code}
$curl http://localhost:8080
Hello World!, v1.0
```

Our initial deployment is very successful. But this was a manual deployment. Every time a newer version of the app is released we have to manually download and deploy the application. This is not what we wanted. Our goal for this demo is to build a Continuous Deployment(CD) pipeline for our nodejs based web service running as a docker container.

Let's begin by setting up various stages in the CD pipeline, like we did in the last week's demo.

## Auto deployment strategy

Our auto deployment strategy has changed and is slightly different from the one used in the last week's native app demo. Here is the auto deployment strategy for this demo:

When a new version of our app is available, we'll update the docker-compose.yml file in the GitHub and release a new version of our app in the GitHub. GitHub will trigger a webhook for this release event. We'll listen for this webhook event and automatically do a git pull on the docker-compose.yml file. We'll use the new docker-compose.yml to re-spin our nodejs web service docker container in production.

**Note:** GitHub will trigger webhooks for all sorts of push events. We don't want to react to all those webhooks and auto deploy our containers on every GitHub push. We want to auto deploy only when a new release is made.

## SocketXP Webhook Relay Service

We'll use **SocketXP Webhook Relay Service** to relay webhooks from GitHub to our local production server (where the nodejs app is running as a docker container). SocketXP can also filter GitHub webhooks based on user specified rules in JSON format. We'll discuss more about it in the next section. Additionally, SocketXP can execute a script or a command, if an incoming webhook matches with the user specified rules.

Before creating webhook rules, it is better to study the webhook in detail and pick up a few fields in the webhook payload to match on. For this we need to make GitHub trigger a release webhook and capture the webhook and study it.

## Webhook Capture and Study

First, let's setup the SocketXP agent to relay any webhooks from the internet to our local web service.

``` {.source-code}
$ socketxp relay http://localhost:8080

Connected.
Public URL -> https://webhook.socketxp.com/ganeshvelrajan-1awy5t0h
```

Pick up the above SocketXP Public URL and head to your GitHub repository. Setup the webhook settings for your GitHub project as shown below. Setup your secret and copy paste the SocketXP Public URL in the webhook URL text box. Also set the content-type to "application/json" in the drop-down box. Save the changes.

![docker-compose-auto-deploy-github-webhook](https://www.socketxp.com/wp-content/uploads/2020/07/docker-compose-auto-deploy-github-webhook.png){.aligncenter .wp-image-1162 .size-full width="1024" height="709"}

## Triggering GitHub Release Webhook

Next, create a new release for your GitHub project in the release settings page and create a new tag for your release, as shown below.

![docker-compose-auto-update-github-webhook-release-v1.0](https://www.socketxp.com/wp-content/uploads/2020/07/docker-compose-auto-update-github-webhook-release.png){.aligncenter .wp-image-1164 .size-full width="1089" height="539"}

GitHub will trigger a webhook for this release event in your project. We can view details of the triggered webhook either in the GitHub Webhook Settings page or in the SocketXP Portal Webhook Logs page.

``` {.source-code}
Request URL: https://webhook.socketxp.com/ganeshvelrajan-1awy5t0h
Request method: POST
Accept: */*
content-type: application/json
User-Agent: GitHub-Hookshot/e509757
X-GitHub-Delivery: ca631384-bd0e-11ea-8ba6-ea05e17356b6
X-GitHub-Event: push
X-Hub-Signature: sha1=c902f69547901dc0658ba54eaee5cde6c0a5ef00

{
  "ref": "refs/tags/v1.0",
  "before": "0000000000000000000000000000000000000000",
  "after": "b5595d43b2c681257e0bae638f9de814df968901",
  "repository": {
    "id": 276817731,
    "node_id": "MDEwOlJlcG9zaXRvcnkyNzY4MTc3MzE=",
    "name": "docker-compose-autoupdate-on-github-webhook",
    "full_name": "socketxp-com/docker-compose-autoupdate-on-github-webhook",
    "private": false,
    ...
    ...
}
```

In the above webhook capture, we see the signature field in the header that encodes the secret we set in the GitHub webhook settings page. Verifying this signature using the secrete we provided, authenticates the validity of the sender.

Secondly, in the above webhook capture, the payload contains the release tags ("refs/tags/v1.0") in the "ref" field.

Let's try to filter our incoming webhook based on these two fields in the webhook.

## SocketXP Webhook Filter-Rules JSON File

Here is how the SocketXP filter-rules JSON file should look like to match the two fields we picked up in the webhook above.

``` {.source-code}
$cat filter-rules.json 
{
  "and":
  [
    {
      "match":
      {
        "type": "payload-hash-sha1",
        "secret": "asdfasdf23421dsaf",    
        "parameter":
        {
          "source": "header",
          "name": "X-Hub-Signature"
        }
      }
    },
    {
      "match":
      {
        "type": "value",
        "value": "refs/tags",
        "parameter":
        {
          "source": "payload",
          "name": "ref"
        }
      }
    }
  ]
}
```

Make sure you update the "secret" field in the filter-rules file above with your GitHub webhook secret.

## SocketXP Exec Command Option

Here is the script we'd like to execute to automatically update our Docker containers running in our production or test server when a new release is made in the GitHub project.

``` {.source-code}
$cat update-docker.sh 
#!/bin/bash

git pull
docker-compose up -d
```

Let's kill our SocketXP agent and re-run it with the two new arguments: "filter-rules.json" file and the "update-docker.sh" script file, as shown below.

``` {.source-code}
socketxp relay http://localhost:8080 --exec update-docker.sh --filter filter-rules.json

Connected.
Public URL -> https://webhook.socketxp.com/ganeshvelrajan-1awy5t0h
```

Note that the SocketXP Webhook Public URL has not changed despite killing and re-executing the command. This is because we are requesting a public URL for a webhook relay service again and that too for the same local service endpoints (http://localhost:8080). This nice feature of SocektXP saves us from updating the GitHub Webhook URL settings again.

## Begin Demo

Now that the CD pipeline is all set, let's upgrade our NodeJS web app and release a version 2.0 of the artifact. We also need to update the docker-compose.yml file to reflect the updated app version.

### App Upgrade

Here is a new version of our app (for the sake of this demo, we have simply updated the version string printed to 2.0).

``` {.source-code}
$cat http-server.js 
var http = require('http');
var port = 8080
var handleRequest = function(request, response) {
  console.log('Received HTTP request for URL: ' + request.url);
  response.writeHead(200);
  response.end('Hello World!, v2.0');
};
var server = http.createServer(handleRequest);
server.listen(port);
```

### Build Push New App Container

Build a new docker container containing this new version of our app.

``` {.source-code}
$docker build -t gvelrajan/http-server:v2.0 .
Sending build context to Docker daemon  69.63kB
Step 1/7 : FROM alpine:latest
 ---> f70734b6a266
Step 2/7 : RUN apk update && apk add nodejs
...
...
Successfully tagged gvelrajan/http-server:v2.0
```

Push the docker image to DockerHub.

``` {.source-code}
$docker push gvelrajan/http-server:v2.0
The push refers to repository [docker.io/gvelrajan/http-server]
4a303822279a: Pushed 
947da554eff8: Layer already exists 
cc403a71dcfb: Layer already exists 
3e207b409db3: Layer already exists 
v2.0: digest: sha256:2a9ee82f76758758c77659260091380b20e3b3d095efdc6480d66e47c5842476 size: 1154
```

Update the docker-compose file with the new docker image version.

``` {.source-code}
$cat docker-compose.yml 
version: '3'
services:
  web:
    image: "gvelrajan/http-server:v2.0"
    ports:
      - "8080:8080"
```

### Git Commit and Release Webhook

Perform git commit and push the changes to the GitHub repository. Create new release version  tag "version 2.0" for your project in GitHub.

![docker-compose-auto-update-github-webhook-release-v2.0](https://www.socketxp.com/wp-content/uploads/2020/07/docker-compose-auto-update-github-webhook-release-2.png){.aligncenter .wp-image-1165 .size-full width="1972" height="1608"}

This event will make GitHub trigger a webhook to SocketXP agent.

### Verify Webhook Received

Check the console where SocketXP agent is running. It should have received the webhook from GitHub and applied the filters on it.

``` {.source-code}
$ socketxp relay http://localhost:8080 --exec update-docker.sh --filter filter-rules.json

Connected.
Public URL -> https://webhook.socketxp.com/ganeshvelrajan-1awy5t0h

Webhook received:
POST / HTTP/1.1
Host: webhook.socketxp.com
Accept: */*
content-type: application/json
User-Agent: GitHub-Hookshot/e509757
X-GitHub-Delivery: fa44ca8e-bd31-11ea-9def-3e0cd2844448
X-GitHub-Event: push
X-Hub-Signature: sha1=d32c79c8de3282e07400711c4e7d3d6542ef79eb

{
  "ref": "refs/tags/2.0",
  "before": "0000000000000000000000000000000000000000",
  "after": "b5595d43b2c681257e0bae638f9de814df968901",
  "repository": {
    "id": 276817731,
    "node_id": "MDEwOlJlcG9zaXRvcnkyNzY4MTc3MzE=",
    "name": "docker-compose-autoupdate-on-github-webhook",
    "full_name": "socketxp-com/docker-compose-autoupdate-on-github-webhook",
    "private": false,
    ...
    ...
}

#Rule: Webhook signature matched.
#Rule: ref in payload matched.
All rules matched.

Executing command: [update-docker.sh]
...
...
```

Check if the docker container has been upgraded and running version 2.0.

``` {.source-code}
$docker ps
CONTAINER ID        IMAGE                        COMMAND                 CREATED              STATUS              PORTS                    NAMES
e7ef0e98cd45        gvelrajan/http-server:v2.0   "node http-server.js"   About a minute ago   Up About a minute   0.0.0.0:8080->8080/tcp   docker-compose-autoupdate-on-github-webhook_web_1_64dc034e55ac
```

### Verify the upgraded app

Curl the web service and verify if the app has been upgraded.

``` {.source-code}
$curl http://localhost:8080
Hello World!, v2.0
```

Our web app prints "v2.0", denoting that it got updated, automatically!

## Conclusion

Setting up Docker Compose to auto deploy on GitHub push or release webhook notification is a great way to set up Continuous Deployment(CD) pipeline. The challenge is in receiving, processing and filtering GitHub webhooks in the production servers or in the internal test servers. SocketXP is a great tool to receive, store, forward, filter and take actions based on webhooks. SocketXP Webhook Relay Service takes care of bulk of the work involved in setting up an automated CD pipeline. So that, developers can focus on their core work, while leaving the rest to SocketXP. Moreover, SocketXP has a free tier for developers. And our paid plans start at just \$1.99/month.
