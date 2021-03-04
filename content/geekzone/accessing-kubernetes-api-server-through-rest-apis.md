---
title: Accessing Kubernetes API Server Through REST API's
description: ""
author: Ganesh Velrajan
tags: [
    Container, Docker, Kubernetes, API
]
date: 2018-10-29
categories: [
    GeekZone, Kubernetes
]
images: ["/images/kubernetes/kubernetes.jpg"]
---

Typically, we use Kubectl CLI utility to talk to the Kubernetes API server to create, update, delete, read any Kubernetes objects like Pods, Deployments, Services etc.

In this tutorial, I'll show you how to directly access the Kubernetes API server using the REST API's.

Kubernetes API server can be accessed directly using the REST API's in two different ways.

## Method \#1 - Kubectl Proxy Mode

In the method, we'll configure Kubectl to run in a Proxy Mode, so that it will authenticate with the API Server.  We can just provide the API to the Kubectl and it will in turn forward our request to the API Server using the appropriate security tokens.

    minikube:~$ kubectl proxy --port=8080
    Starting to serve on 127.0.0.1:8080

The above CLI is a blocking CLI call.  So it will run until your kill it using Ctrl+C

Next point the Curl web tool to this kubectl proxy server.  The kubectl inturn will redirect our API requests to the Kubernetes API server.

    minikube:~$ curl localhost:8080/api
    {
      "kind": "APIVersions",
      "versions": [
        "v1"
      ],
      "serverAddressByClientCIDRs": [
        {
          "clientCIDR": "0.0.0.0/0",
          "serverAddress": "10.160.0.3:8443"
        }
      ]
    }
    minikube:~$

Now, if you want to access any object within the Kubernetes Cluster then you need to get the appropriate REST API information from the [Kubernetes API Reference guide](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.12/).  They keep changing the location of the Kubernetes API Reference guide , frequently.  So if the above link is broken, you need to Google Search to find it again.

In the above guide, I looked at the section on Pod.  I'm interested in reading the contents of a specific pod that I have created in the Kubernetes Cluster.

![Kubernetes API Reference Guide](http://www.ethernetresearch.com/wp-content/uploads/2018/10/Kubernetes-API-Reference-Guide.jpg){.aligncenter .wp-image-1216 .size-full width="992" height="768"}

The API we need to use is the following:

    /api/v1/namespaces/{namespace-name}/pods/{pod-name}

Now let's use Curl to access this API.

    minikube:~$ curl localhost:8080/api/v1/namespaces/default/pods/helloworld

    {
      "kind": "Pod",
      "apiVersion": "v1",
      "metadata": {
        "name": "helloworld",
        "namespace": "default",
        "selfLink": "/api/v1/namespaces/default/pods/helloworld",
        "uid": "f1857174-db41-11e8-ab3f-42010aa00003",
        "resourceVersion": "11847",
        "creationTimestamp": "2018-10-29T06:14:50Z",
        "labels": {
          "app": "helloworld"
        }
      },
      "spec": {
        "volumes": [
          {
            "name": "default-token-htj76",
            "secret": {
              "secretName": "default-token-htj76",
              "defaultMode": 420
            }
          }
        ],
        "containers": [
          {
            "name": "helloworld",
            "image": "gvelrajan/helloworld:v2.0",
            "ports": [
              {
                "containerPort": 80,
                "protocol": "TCP"
              }
            ],
            "resources": {
              
            },
            "volumeMounts": [
              {
                "name": "default-token-htj76",
                "readOnly": true,
                "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount"
              }
            ],
            "terminationMessagePath": "/dev/termination-log",
            "terminationMessagePolicy": "File",
            "imagePullPolicy": "Always"
          }
        ],
        "restartPolicy": "Always",
        "terminationGracePeriodSeconds": 30,
        "dnsPolicy": "ClusterFirst",
        "serviceAccountName": "default",
        "serviceAccount": "default",
        "nodeName": "minikube",
        "securityContext": {
          
        },
        "schedulerName": "default-scheduler",
        "tolerations": [
          {
            "key": "node.kubernetes.io/not-ready",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          },
          {
            "key": "node.kubernetes.io/unreachable",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          }
        ]
      },
      "status": {
        "phase": "Running",
        "conditions": [
          {
            "type": "Initialized",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2018-10-29T06:14:50Z"
          },
          {
            "type": "Ready",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2018-10-29T06:14:55Z"
          },
          {
            "type": "PodScheduled",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2018-10-29T06:14:50Z"
          }
        ],
        "hostIP": "10.160.0.3",
        "podIP": "172.17.0.2",
        "startTime": "2018-10-29T06:14:50Z",
        "containerStatuses": [
          {
            "name": "helloworld",
            "state": {
              "running": {
                "startedAt": "2018-10-29T06:14:55Z"
              }
            },
            "lastState": {
              
            },
            "ready": true,
            "restartCount": 0,
            "image": "gvelrajan/helloworld:v2.0",
            "imageID": "docker-pullable://gvelrajan/helloworld@sha256:897f0c9ec9f26b32a258f027bde0d276954dc7bf666d34cc85106cb3430c03de",
            "containerID": "docker://d9751038d20c827eac9bcfa94b71256b494ac442b2822b74a51312fd1ed7653d"
          }
        ],
        "qosClass": "BestEffort"
      }
    }

We see that our "helloworld" Pod, like any Kubernetes object, has a "Spec" which is the user expected state(intention) of the object and "Status" which is the current state of the object.

As a developer or a DevOps Engineer, you can use Kubectl as a proxy to access the API server directly via the REST API's.

## Method \#2 - Direct Access using Tokens and Certificates

In this method, we'll use the Web Request Tools such as Curl to directly authenticate  with the API server.

First get the API server's IP address and TCP port using the following command.

    minikube:~$ kubectl config view
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority: /home/gannygans/.minikube/ca.crt
        server: https://10.160.0.3:8443
      name: minikube
    contexts:
    - context:
        cluster: minikube
        user: minikube
      name: minikube
    current-context: minikube
    kind: Config
    preferences: {}
    users:
    - name: minikube
      user:
        client-certificate: /home/gannygans/.minikube/client.crt
        client-key: /home/gannygans/.minikube/client.key
    minikube:~$ 

Next we need to get the secrete token for authentication.  We'll use the following two command to get the information.

    $ kubectl get secrets
    $ kubectl describe secret <secret-name>

We'll use the following single line shell script to retrieve the token from the above commands and set it in an environmental variable named "TOKEN".

    minikube:~$ TOKEN=$(kubectl describe secret $(kubectl get secrets | grep ^default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d " ")

    minikube:~$ echo $TOKEN
    eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW
    50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia
    3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4t
    aHRqNzYiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1
    lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW
    50LnVpZCI6IjM0MDU1MTQ1LWQwZjktMTFlOC05NDk0LTQyMDEwYWEwMDAwMyIsInN1YiI6InN5c
    3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.JR1mrMxaaW1Ty22bPGh9M1PVY
    1wRi85ZrDz5DYztkqhO8zaqcmUwdWeOaKtdiA4fHcAx7CRupLRXWxCWsVZ4KK-sRDq6CTNI6knz
    LVvm1n0fjLLt15mY8bWdTZld1r2hKS5NEcE05XDCFOAnmjFJuHxtUq1dD4fhUM2g_EuZgZL44CK
    nTtdWWmhTBNK4ejghWQeW3M7XgewS_J3-URqSUzCkWAWMAfrPYsKb_70kecm_tQuukOL72ZWbRL
    333xLp5GVzXzHjBhBf5CPYOXGT-OXd4IvnkZ7vhE3kf2lE8K-fKnOjIauHl2pLFN5LapqM6IMWG
    zoV6NmEv_k6E7x-Yw

Let's use the above token in the curl command to directly access the Kubernetes API server, as shown below.

    minikube:~$ curl https://10.160.0.3:8443/api/namespaces/default/pods/helloworld --header "Authorization: Bearer $TOKEN" --insecure

    {
      "kind": "Pod",
      "apiVersion": "v1",
      "metadata": {
        "name": "helloworld",
        "namespace": "default",
        "selfLink": "/api/v1/namespaces/default/pods/helloworld",
        "uid": "f1857174-db41-11e8-ab3f-42010aa00003",
        "resourceVersion": "11847",
        "creationTimestamp": "2018-10-29T06:14:50Z",
        "labels": {
          "app": "helloworld"
        }
      },
      "spec": {
        "volumes": [
          {
            "name": "default-token-htj76",
            "secret": {
              "secretName": "default-token-htj76",
              "defaultMode": 420
            }
          }
        ],
        "containers": [
          {
            "name": "helloworld",
            "image": "gvelrajan/helloworld:v2.0",
            "ports": [
              {
                "containerPort": 80,
                "protocol": "TCP"
              }
            ],
            "resources": {
              
            },
            "volumeMounts": [
              {
                "name": "default-token-htj76",
                "readOnly": true,
                "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount"
              }
            ],
            "terminationMessagePath": "/dev/termination-log",
            "terminationMessagePolicy": "File",
            "imagePullPolicy": "Always"
          }
        ],
        "restartPolicy": "Always",
        "terminationGracePeriodSeconds": 30,
        "dnsPolicy": "ClusterFirst",
        "serviceAccountName": "default",
        "serviceAccount": "default",
        "nodeName": "minikube",
        "securityContext": {
          
        },
        "schedulerName": "default-scheduler",
        "tolerations": [
          {
            "key": "node.kubernetes.io/not-ready",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          },
          {
            "key": "node.kubernetes.io/unreachable",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          }
        ]
      },
      "status": {
        "phase": "Running",
        "conditions": [
          {
            "type": "Initialized",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2018-10-29T06:14:50Z"
          },
          {
            "type": "Ready",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2018-10-29T06:14:55Z"
          },
          {
            "type": "PodScheduled",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2018-10-29T06:14:50Z"
          }
        ],
        "hostIP": "10.160.0.3",
        "podIP": "172.17.0.2",
        "startTime": "2018-10-29T06:14:50Z",
        "containerStatuses": [
          {
            "name": "helloworld",
            "state": {
              "running": {
                "startedAt": "2018-10-29T06:14:55Z"
              }
            },
            "lastState": {
              
            },
            "ready": true,
            "restartCount": 0,
            "image": "gvelrajan/helloworld:v2.0",
            "imageID": "docker-pullable://gvelrajan/helloworld@sha256:897f0c9ec9f26b32a258f027bde0d276954dc7bf666d34cc85106cb3430c03de",
            "containerID": "docker://d9751038d20c827eac9bcfa94b71256b494ac442b2822b74a51312fd1ed7653d"
          }
        ],
        "qosClass": "BestEffort"
      }
    }

Again, we get the same output from the API server.

To setup Authentication, Authorization and Admission Control appropriately for accessing the API Server, refer to this article in Kubernetes Documentation.

[Controlling Access to the Kubernetes API Server](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/)

That's all folks !  Hope you got some idea on how to talk to the Kubernetes API Server directly, without using any Kubectl commands.
