---
title: Kubernetes Security - User Authentication and Authorization (RBAC)
description: ""
author: Ganesh Velrajan
tags: [
    Container, Docker, Kubernetes, Security
]
date: 2018-11-01
categories: [
    GeekZone, Kubernetes
]
images: ["/images/kubernetes/kubernetes.jpg"]
---

In the article, we'll discuss how to create credentials and add a new user to the Kubernetes Cluster to perform various roles in the cluster.

To add a new user, we as an Admin, should create and approve the SSL Private Keys and Certificate for the user using the Kubernetes Certificate Manager.

So, let's get started.

## Authentication:

### Create Private Key

First, let's create a private key for the user, in a separate folder.

    minikube:~$ mkdir credentials
    minikube:~$ cd credentials/

    minikube:~/credentials$ openssl genrsa -out ganesh.key 2048
    Generating RSA private key, 2048 bit long modulus
    ...........................................................................
    ................+++
    ...........................................................................
    ....+++
    e is 65537 (0x10001)
    minikube:~/credentials$

### Create Certificate Signing Request

Next, let's create the Certificate Signing Request (CSR) for the private key we generated above.

    minikube:~/credentials$ openssl req -new -key ganesh.key -out ganesh.csr -subj "/CN=ganesh/O=EthernetResearch"\n
    minikube:~/credentials$
    minikube:~/credentials$ ls 
    ganesh.csr ganesh.key
    minikube:~/credentials$

Now let's encode the CSR into a base64 format.

    minikube:~/credentials$ cat ganesh.csr | base64
    LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ2REQ0NBVndDQVFBd0x6RVJNQThHQTFVRUF3d0laM1psYkhKaGFtRXhHakFZQmdOVkJBb01FVVYwYUdWeQpibVYwVW1WelpXRnlZMmh1TUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUExNTNGClNzaWRCdzF5cUZhSGdWblJjdmJYNjhzZUNSYm9KbEVOY1BSUFkySEFwcVJJaWVUZW9ESGpPWktWOExsZkFoYlgKUTNVYXJCSUxVcHd2UkVwUy9iWnFJM0d6eGxHbnYzOE9iZGx0azRjYW8xK0xnTnEzaDZIWUlpUklwWllzUDVCYgoybFFyWDdsNkR4dktaOTdrdThtRUR1cmdUb3o4NzlZNll5ZVJobUlaOXdkK2RUSjMvZEo0VUl0RG44YWl3MUhuCm5SMXE4cDdYd205YjVEU1YxNnAzdENZTFF6YlpxUCt0UEd4N0VKU05hSTNZNkkzamhDcmYrYUtvZGJjU1dBR0YKTUJxVjJkajY4N2RxbU42WEdraTRCbCtZd2VaV3doTy9Gb2YxZTJuWlZVd0Q1aVRlRkFmK0h6WGJibFhpK0RCMwpMd2QyWm85QU1lblVQZTZHQ3dJREFRQUJvQUF3RFFZSktvWklodmNOQVFFTEJRQURnZ0VCQUY3VDBHNU5tVy9zCm5UTTVPWitmcC84K3BDb0xPblNqbHk3TC9XdVRwWjdNUi83MjRGZE1ObWtUUHZCc0ZqTHhEUGhMaXhpWk5LcEQKYVc3RGxNVWw1a1FydzlnOGdXTnd2S1BkMmV4OGpQb2ErZENpbngvejlBdGlHN244OFlveFd1MFJHS0lzWGh5cgpmYnVCdVdaejExTDBhRWl5YU41VXNrOWNZL21IYlN6dERlV01IZ3VUb1hSUFNiU0wxZFJTSThLSlZ4RWV6eWdZCmVKeHdYYnB6VEtteStDT0pHMmgycmszY0t4V1FiY0VESXgyem5zVStOK0lROWw3ZlZIR3R4V0dXVWpZQ0hSUkcKeG5hT1Q3Q3NHM3JEaWVRN1BBR3hoNFZTZ3JnMmlMQStIYklrbHJkdm1mVkNVdGFJa1NYYnpCZGZpYWFSWkJFagowRzRLaGp0bjQ4QT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==

### Submit Certificate Signing Request to K8s

Let's use the base64 encoded CSR in our request to Kubernetes Certificate Manager.

    minikube:~/credentials$ cat signing-request.yaml 
    apiVersion: certificates.k8s.io/v1beta1
    kind: CertificateSigningRequest
    metadata:
      name: ganesh-csr
    spec:
      groups:
      - system:authenticated
      request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ2REQ0NBVndDQVFBd0x6RVJNQThHQTFVRUF3d0laM1psYkhKaGFtRXhHakFZQmdOVkJBb01FVVYwYUdWeQpibVYwVW1WelpXRnlZMmh1TUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUExNTNGClNzaWRCdzF5cUZhSGdWblJjdmJYNjhzZUNSYm9KbEVOY1BSUFkySEFwcVJJaWVUZW9ESGpPWktWOExsZkFoYlgKUTNVYXJCSUxVcHd2UkVwUy9iWnFJM0d6eGxHbnYzOE9iZGx0azRjYW8xK0xnTnEzaDZIWUlpUklwWllzUDVCYgoybFFyWDdsNkR4dktaOTdrdThtRUR1cmdUb3o4NzlZNll5ZVJobUlaOXdkK2RUSjMvZEo0VUl0RG44YWl3MUhuCm5SMXE4cDdYd205YjVEU1YxNnAzdENZTFF6YlpxUCt0UEd4N0VKU05hSTNZNkkzamhDcmYrYUtvZGJjU1dBR0YKTUJxVjJkajY4N2RxbU42WEdraTRCbCtZd2VaV3doTy9Gb2YxZTJuWlZVd0Q1aVRlRkFmK0h6WGJibFhpK0RCMwpMd2QyWm85QU1lblVQZTZHQ3dJREFRQUJvQUF3RFFZSktvWklodmNOQVFFTEJRQURnZ0VCQUY3VDBHNU5tVy9zCm5UTTVPWitmcC84K3BDb0xPblNqbHk3TC9XdVRwWjdNUi83MjRGZE1ObWtUUHZCc0ZqTHhEUGhMaXhpWk5LcEQKYVc3RGxNVWw1a1FydzlnOGdXTnd2S1BkMmV4OGpQb2ErZENpbngvejlBdGlHN244OFlveFd1MFJHS0lzWGh5cgpmYnVCdVdaejExTDBhRWl5YU41VXNrOWNZL21IYlN6dERlV01IZ3VUb1hSUFNiU0wxZFJTSThLSlZ4RWV6eWdZCmVKeHdYYnB6VEtteStDT0pHMmgycmszY0t4V1FiY0VESXgyem5zVStOK0lROWw3ZlZIR3R4V0dXVWpZQ0hSUkcKeG5hT1Q3Q3NHM3JEaWVRN1BBR3hoNFZTZ3JnMmlMQStIYklrbHJkdm1mVkNVdGFJa1NYYnpCZGZpYWFSWkJFagowRzRLaGp0bjQ4QT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==

      usages:
      - digital signature
      - key encipherment
      - server auth

Next let's submit the request to Kubernetes Certificate Manager.

    minikube:~/credentials$ kubectl create -f signing-request.yaml certificatesigningrequest.certificates.k8s.io/ganesh-csr created

    minikube:~/credentials$ kubectl get csr
    NAME AGE REQUESTOR CONDITION
    ganesh-csr 1m minikube-user Pending
    minikube:~/credentials$

### Approve the Certificate as K8s Admin

Next, as a Kubernetes Admin, let's approve the Certificate Signing Request.

    minikube:~/credentials$ kubectl certificate approve ganesh-csr
    certificatesigningrequest.certificates.k8s.io/ganesh-csr approved
    minikube:~/credentials$

    minikube:~/credentials$ kubectl get csr
    NAME AGE REQUESTOR CONDITION
    ganesh-csr 6m minikube-user Approved,Issued
    minikube:~/credentials$

Now the user certificated has been successfully  approved, let's retrieve the signed Certificate and store it in a local file named "ganesh.crt".

    minikube:~/credentials$ kubectl get csr ganesh-csr -o jsonpath='{.status.certificate}' | base64 --decode > ganesh.crt

    minikube:~/credentials$ ls
    ganesh.crt ganesh.csr ganesh.key signing-request.yaml

    minikube:~/credentials$ cat ganesh.crt 
    -----BEGIN CERTIFICATE-----
    MIIDJDCCAgygAwIBAgIUWea2B/T52AUW1yUpfqfDxFnM7FkwDQYJKoZIhvcNAQEL
    BQAwFTETMBEGA1UEAxMKbWluaWt1YmVDQTAeFw0xODExMDEwNjQ0MDBaFw0xOTEx
    MDEwNjQ0MDBaMC0xGjAYBgNVBAoTEUV0aGVybmV0UmVzZWFyY2huMQ8wDQYDVQQD
    EwZnYW5lc2gwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCoNnKewPY8
    ABlrPaJi2zQ43HgvgA5jZse2qAk4S2L4HfVGci/A8k+t62zhcQj9uMhwR5yhWX+8
    98nvyDumD8B6my0YIMcfslc4liGcZSlwBnrQDSvin5TtcsRJDXE4fLuOrbuAWpP5
    RXDFK+AVj2wAkmis4i8dMNL+X63B1Kng9Wfj/eboM6JSl4kLoTrCM6dtHlvwAxcc
    1u8N2ceYsVof4rmF7Tjzn4YjG1j6+ZKI5dgaxl2sIxtrtEOBO4oadnhTfdFDUkLn
    DSIb51ZyWnKPp3kfPTno00coXhhM6LFyiTMX0H3a/yH1oQVb/MZONbyo8R2JhUXB
    HhRuHeCvIlh/AgMBAAGjVDBSMA4GA1UdDwEB/wQEAwIFoDATBgNVHSUEDDAKBggr
    BgEFBQcDATAMBgNVHRMBAf8EAjAAMB0GA1UdDgQWBBQe/8B6PROMLku2VdNFVvLe
    QT2PXTANBgkqhkiG9w0BAQsFAAOCAQEAYzjYxH05MlEfC8Oue7szI4pzL/iGRQA+
    u9BYggbrY5STGHN64yB0sgkzD3d+nmKF1Vl/O4CKzxr1WgEl0ENxuj8ZdfTJy+7R
    MjJPvc77LSP4378Osw1Y38EEpmgm/g611XA8V5zbXMSJtb+N3aghgdDRmyQxC0WF
    2aW59MEE0mnd2/t95RxaJhxAV7i2NCz8XI9iW7GigMl6+PYWIl5+g8RBzUnhUMiF
    uU3vM9ELIbdPVtdz5UPSYQIZR/1o6coB8iDvGZbConJazn8E/4Surh7iKKtXvOGG
    STWLpS8Rh5XhIofkYS2XDTBvNCc+C8WrvPFalPKrrFI/Mux2SayY2Q==
    -----END CERTIFICATE-----

### Create New User in K8s Using the Credentials

Now, let's create the user "ganesh" in Kubernetes with the approved credentials we have just generated.

    minikube:~/credentials$ kubectl config set-credentials ganesh --client-certificate=ganesh.crt --client-key=ganesh.key
    User "ganesh" set.

    minikube:~/credentials$ kubectl config view
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority: /home/gannygans/.minikube/ca.crt
        server: https://10.148.0.2:8443
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
    - name: ganesh
      user:
        client-certificate: /home/gannygans/credentials/ganesh.crt
        client-key: /home/gannygans/credentials/ganesh.key
    - name: minikube
      user:
        client-certificate: /home/gannygans/.minikube/client.crt
        client-key: /home/gannygans/.minikube/client.key
    minikube:~/credentials$ 

### Create a New Namespace in the Cluster

Now, let's create a new namespace in the cluster and assign the user "ganesh" with access rights to that namespace.

    minikube:~$ kubectl create namespace finance
    namespace/finance created
    minikube:~$ kubectl get namespace
    NAME STATUS AGE
    default Active 4h
    finance Active 6s
    kube-public Active 4h
    kube-system Active 4h
    minikube:~$

### Create New Context

Next, let's create a new context and associate the user with that context.

Context is very useful if we have many Kubernetes clusters to manage and wanted to switch the context from one Kubernetes Cluster to another, so that you could execute commands related to that cluster.  Context makes the job of switching the connection very easy.

Context is also very useful if you want to switch from one namespace in a cluster to another namespace in the same cluster or in a different cluster.

    minikube:~/credentials$ kubectl config set-context finance-context --cluster=minikube --namespace=finance --user=ganesh
    Context "finance-context" created.

    minikube:~/credentials$ kubectl config view
    apiVersion: v1
    clusters:
    - cluster:
    certificate-authority: /home/gannygans/.minikube/ca.crt
    server: https://10.148.0.2:8443
    name: minikube
    contexts:
    - context:
    cluster: minikube
    namespace: finance
    user: ganesh
    name: finance-context
    - context:
    cluster: minikube
    user: minikube
    name: minikube
    current-context: minikube
    kind: Config
    preferences: {}
    users:
    - name: ganesh
    user:
    client-certificate: /home/gannygans/credentials/ganesh.crt
    client-key: /home/gannygans/credentials/ganesh.key
    - name: minikube
    user:
    client-certificate: /home/gannygans/.minikube/client.crt
    client-key: /home/gannygans/.minikube/client.key
    minikube:~/credentials$

### Check User Previleges:

Now, let's check the access previleges of the user "ganesh"  we have just created.

    minikube:~$ kubectl auth can-i list pods --namespace finance
    yes
    minikube:~$ kubectl auth can-i list pods --namespace finance --as ganesh
    no

We see that as an admin we can access pods in "finance" namespace but not as user "ganesh". Let's try accessing more resources in the "finance" namespace.

Let's try to create a simple helloworld pod, as we did in the previous demos. Let's execute the below commands as an admin because the user "ganesh" doesn't have the access rights to create resources in the minikube cluster yet.

    minikube-2:~$ cat pod.yaml
    apiVersion: v1
    kind: Pod                                            # 1
    metadata:
      name: helloworld                                   # 2
      namespace: finance
      labels:                                            
        app: helloworld
    spec:                                                # 3
      containers:
        - image: gvelrajan/helloworld:v2.0              # 4
          imagePullPolicy: Always
          name: helloworld                               # 5
          ports:
            - containerPort: 80                          # 6

    minikube:~$ kubectl create -f pod.yaml
    pod/helloworld created

    minikube:~$ kubectl get pods --namespace=finance
    NAME         READY   STATUS    RESTARTS   AGE
    helloworld   1/1     Running   0          10s

    minikube:~$ kubectl get pods --namespace finance --as ganesh
    Error from server (Forbidden): pods is forbidden: User "ganesh" cannot list pods in the namespace "finance"

Let's also check if the user "ganesh" has access to cluster resources like "nodes" that are not bound by a namespace.

    minikube:~$ kubectl get nodes --namespace finance 
    NAME STATUS ROLES AGE VERSION
    minikube Ready master 1d v1.10.0
    minikube:~$ kubectl get nodes --namespace finance --as ganesh
    Error from server (Forbidden): nodes is forbidden: User "ganesh" cannot list nodes at the cluster scope
    minikube:~$

By default, a user doesn't have any access rights to any resources.  The admin needs to authorize the user to perform various actions or roles within the cluster or a in a namespace within the cluster.

## Authorization:

### Role Based Access Control (RBAC)

Next, I'll show you how to authorize the user "ganesh" to perform various roles in a specific namespace or in the entire cluster using RBAC .

    minikube-2:~$ cat role.yaml 
    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      namespace: finance 
      name: finance-reader
    rules:
    - apiGroups: [""] # "" indicates the core API group
      resources: ["pods", "services", "nodes"]
      verbs: ["get", "watch", "list"]

Let's create the role using the above yaml file.

    minikube:~$ kubectl create -f role.yaml 
    role.rbac.authorization.k8s.io/finance-reader created
    minikube:~$ kubectl get roles --namespace=finance
    NAME AGE
    finance-reader 56s

Next, we have to bind the role to the user.

    minikube:~$ cat role-binding.yaml 
    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: finance-read-access
      namespace: finance 
    subjects:
    - kind: User
      name: ganesh # Name is case sensitive
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: Role #this must be Role or ClusterRole
      name: finance-reader # this must match the name of the Role or ClusterRole you wish to bind to
      apiGroup: rbac.authorization.k8s.io

    minikube:~$ kubectl create -f role-binding.yaml 
    rolebinding.rbac.authorization.k8s.io/finance-read-access created

    minikube:~$ kubectl get rolebindings --namespace=finance
    NAME AGE
    finance-read-access 42s
    minikube:~$

Let's check if the user "ganesh" has access to pods now.

    minikube:~$ kubectl get pods --namespace=finance --as=ganesh
    NAME READY STATUS RESTARTS AGE
    helloworld 1/1 Running 0 43s

    minikube:~$ kubectl get pods --as=ganesh
    Error from server (Forbidden): pods is forbidden: User "ganesh" cannot list pods in the namespace "default"
    minikube:~$

User "ganesh" has access to pods in the "finance" namespace but not in any other namespace such as the "default" namespace.  This behaviour is as expected.

Next, let's check if the user has access to any cluster resources like "nodes'

    minikube:~$ kubectl get nodes --namespace=finance --as=ganesh
    Error from server (Forbidden): nodes is forbidden: User "ganesh" cannot list nodes at the cluster scope

User "ganesh" doesn't have access to "nodes", despite granting access to it in the above "role.yaml file."  Why ?

This is because "nodes" is a cluster resource and not a resource that belongs to the "finance" namespace or any namespace.  So eventhough we had mentoned "nodes" as one of the resources in the  finance namespace "role.yaml" file above.  It semantically doesn't make any sense.  Hence user "ganesh" doesn't get any permission to read cluster nodes.

How do we solve this problem ?

The answer is: "Cluster Role and Cluster Role Binding"

### Create Cluster Role

We need to create a Kubernetes Cluster Role and then bind the Cluster Role to the user "ganesh" to access the Kubernetes cluster nodes.

    minikube:~$ cat cluster-role.yaml 
    kind: ClusterRole
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      # "namespace" omitted since ClusterRoles are not namespaced
      name: cluster-node-reader
    rules:
    - apiGroups: [""]
      resources: ["nodes"]
      verbs: ["get", "watch", "list"]

Let's create the cluster role using the above yaml file.

    minikube:~$ kubectl create -f cluster-role.yaml 
    clusterrole.rbac.authorization.k8s.io/cluster-node-reader created


    minikube:~$ kubectl get clusterroles
    NAME                                                AGE
    admin                                               25h
    cluster-admin                                       25h
    cluster-node-reader                                 19s
    edit                                                25h
    system:aggregate-to-admin                           25h
    ...
    ...

    system:node-problem-detector                        25h
    system:node-proxier                                 25h
    system:persistent-volume-provisioner                25h
    system:volume-scheduler                             25h
    view                                                25h

### Cluter Role Binding

Next, let's use the following Cluster Role Binding to bind the Cluster Role to the user "ganesh".

    minikube:~$ cat cluster-role-binding.yaml 
    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: read-cluster-nodes
    subjects:
    - kind: User
      name: ganesh # Name is case sensitive
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: ClusterRole
      name: cluster-node-reader
      apiGroup: rbac.authorization.k8s.io

    minikube:~$ kubectl create -f cluster-role-binding.yaml 
    clusterrolebinding.rbac.authorization.k8s.io/read-cluster-nodes created
    minikube:~$ kubectl get clusterrolebindings
    NAME                                                   AGE
    cluster-admin                                          26h
    kubeadm:kubelet-bootstrap                              26h
    kubeadm:node-autoapprove-bootstrap                     26h
    kubeadm:node-autoapprove-certificate-rotation          26h
    kubeadm:node-proxier                                   26h
    minikube-rbac                                          26h
    read-cluster-nodes                                     10s
    storage-provisioner                                    26h
    system:aws-cloud-provider                              26h
    system:basic-user                                      26h
    ...
    ...
    system:kube-controller-manager                         26h
    system:kube-dns                                        26h
    system:kube-scheduler                                  26h
    system:node                                            26h
    system:node-proxier                                    26h
    system:volume-scheduler                                26h

### Verify User's Cluster Roles

Now let's check if the user "ganesh" has access to read the cluster nodes.

    minikube:~$ kubectl get nodes --namespace=finance --as=ganesh
    NAME       STATUS   ROLES    AGE   VERSION
    minikube   Ready    master   1d    v1.10.0

We don't have to specify the "--namespace" option because nodes are cluster resources and not bound to any namespace.

    minikube:~$ kubectl get nodes  --as=ganesh
    NAME       STATUS   ROLES    AGE   VERSION
    minikube   Ready    master   1d    v1.10.0

Let's check if user "ganesh" has access to any other cluster resource other than "nodes".

    minikube:~$ kubectl get secrets  --as=ganesh
    Error from server (Forbidden): secrets is forbidden: User "ganesh" cannot list secrets 
    in the namespace "default"

As we expected, user "ganesh" cannot access any other cluster resources except for the "nodes".

That's also folks, I have to discuss today, about security and user authentication, authorization using RBAC.
