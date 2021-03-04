---
title: Kubernetes Dashboard Setup and Login
description: ""
author: Ganesh Velrajan
tags: [
    Container, Docker, Kubernetes, Dashboard
]
date: 2018-10-24
categories: [
    GeekZone, Kubernetes
]
images: ["/images/kubernetes/kubernetes.jpg"]
---

## Deploy the Dashboard UI:

    master$ kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml

We can either directly talk to the API server to connect to the dashboard or if we would like to use the "kubectl" command then we should run the following proxy command. This command is a blocking call.

    master$ kubectl proxy
    Starting to serve on 127.0.0.1:8001

 

## Creating Admin Token for Login:

Now open another terminal window in the master node and execute the following commands to the create the admin token for login to the Dashboard.

Create the service account first using the following YAML file.  You can name the file anything.

    master$ cat service-account.yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: admin-user
      namespace: kube-system

    master$ kubectl create -f service-account.yaml
    serviceaccount/admin-user created

The admin Role already exists in the cluster. We can use it for login. We just need to create only RoleBinding for the ServiceAccount we create above.

    master$ cat role-binding.yaml
    apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: ClusterRoleBinding
    metadata:
      name: admin-user
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - kind: ServiceAccount
      name: admin-user
      namespace: kube-system

    master$ kubectl create -f role-binding.yaml
    clusterrolebinding.rbac.authorization.k8s.io/admin-user created

Now let's find the token we need to use to login.

    master$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
    Name:         admin-user-token-6bphj
    Namespace:    kube-system
    Labels:       
    Annotations:  kubernetes.io/service-account.name=admin-user
                  kubernetes.io/service-account.uid=b472e305-a92c-11e8-85f8-0800277d1239
    Type: kubernetes.io/service-account-token

    Data
    ====
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLTZicGhqIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJiNDcyZTMwNS1hOTJjLTExZTgtODVmOC0wODAwMjc3ZDEyMzkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.TFkjDjoz5crJ0JTupCFSY-qLmoFgxhiy_Js69JALlzs6Uof7Y2CSHpLo_7F_8xiOPlnVvWibXGh6CslXPTH87L-8uYCDpYfYoQBfl0nJjFqP1860IaoJIEdaRYshEvd4RHtZbRiN82zTEDeXMozuwKK3_wQyFf1eZMHcyWtt94KW3_kuGM6DvsdkDM59aKU1LiWif4cRXghnXy0yiEEwcCREwxRqDShzX5I3ne_hN04AnR7DM_r8Tjw0VgTOil7X3UTEZ0BD13tpdgA2K4xb551lQQC6Jr-p5bf1-fLa7sA7ztzxKRZt7rm4A8tDHNve5WPfesOtODPz4L7IAGx1nQ
    ca.crt: 1025 bytes
    namespace: 11 bytes

Now point the browser to the following URL:

http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

Input the token we got from the above output into the token field below.

![Kubernetes Dashboard Login](http://www.thetechzone.in/wp-content/uploads/2018/09/K8-Dashboard1.jpg){.wp-image-48 .size-full width="640" height="276"} 
**Kubernetes Dashboard Login Screen**

After we login, we can access the pods and deployments in the Kubernetes dashboard.

![Kubernetes Dashboard Setup](http://www.thetechzone.in/wp-content/uploads/2018/09/K8-Dashboard2.jpg)
**Kubernetes Dashboard**
