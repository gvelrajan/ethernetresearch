---
title: How to install Jenkins on Ubuntu 20.04 LTS
description: ""
author: Ganesh Velrajan
tags: [
    Jenkins, CI/CD, Linux, Ubuntu 20.04 LTS
]
date: 2018-11-15
categories: [
    CI/CD, GeekZone
]
images: ["/images/microservice/jenkins.png"]
---

In this tutorial, I'll show you how to install Jenkins on Ubuntu 16.04 and configure Jenkins. You can also refer to the [Jenkins download page](https://pkg.jenkins.io/debian-stable/) to see the instructions.

### Prerequisites:

-   Ubuntu 16.04
-   Java 7 or 8

### Installing Java 8

Java is a prerequisite to run Jenkins.  We need to install Java first.  Jenkins works best with Java version 7 or 8 but not 9.

Use the following command to install Java version 8.

    $ sudo apt install openjdk-8-jre-headless

## Installing Jenkins:

Add the repository key to the local apt package manager.

    $ wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -

Next, we need to append the Debian package repository address to the server's *'sources.list'*

    $ echo deb https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list

Next, update the package manager so that apt-get will use newly added repository.

    $ sudo apt-get update

Now install Jenkins.

    $ sudo apt-get install jenkins

 

## Starting Jenkins:

    $ sudo systemctl start jenkins

Check the status of the Jenkins using the below command.

    $ sudo systemctl status jenkins
    ● jenkins.service - LSB: Start Jenkins at boot time
       Loaded: loaded (/etc/init.d/jenkins; bad; vendor preset: enabled)
       Active: active (exited) since Thu 2018-11-15 05:55:29 UTC; 40min ago
         Docs: man:systemd-sysv-generator(8)
      Process: 6353 ExecStart=/etc/init.d/jenkins start (code=exited, status=0/SUCCES
        Tasks: 0
       Memory: 0B
          CPU: 0
    Nov 15 05:55:28 jenkins systemd[1]: Starting LSB: Start Jenkins at boot time...
    Nov 15 05:55:28 jenkins jenkins[6353]: Correct java version found
    Nov 15 05:55:28 jenkins jenkins[6353]:  * Starting Jenkins Automation Server jenk
    Nov 15 05:55:28 jenkins su[6399]: Successful su for jenkins by root
    Nov 15 05:55:28 jenkins su[6399]: + ??? root:jenkins
    Nov 15 05:55:28 jenkins su[6399]: pam_unix(su:session): session opened for user j
    Nov 15 05:55:29 jenkins jenkins[6353]:    ...done.
    Nov 15 05:55:29 jenkins systemd[1]: Started LSB: Start Jenkins at boot time.
    Nov 15 05:59:13 jenkins systemd[1]: Started LSB: Start Jenkins at boot time.

 

## Configuring Jenkins:

Jenkins runs on port 8080 on the localhost.  Use your browser to connect to Jenkins using the below URL

    http://localhost:8080/

**Note**:  If you are installing Jenkins in a Virtual Machine in a cloud, then use the public IP address of the Virtual Machine provided by your cloud service provider.  Also make sure you have updated the cloud provider's firewall rule setting to allow traffic from external sources to port 8080.

After you connect to the Jenkins server, the browser will display an "Unlock" Jenkins page as shown below.

![unlock-jenkins : How to install Jenkins on Ubuntu](http://www.ethernetresearch.com/wp-content/uploads/2018/11/unlock-jenkins.jpg){.aligncenter .wp-image-1241 .size-full width="1137" height="590"}

Use the below command to retrieve the key to unlock Jenkins.

    $ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    b6e2d139844c47f79dad47285cd3998d
    $

In the next page, choose the option to "install the suggested plugins".

![jenkins-customize : How to install Jenkins on Ubuntu](http://www.ethernetresearch.com/wp-content/uploads/2018/11/jenkins-customize.jpg){.aligncenter .wp-image-1238 .size-full width="1137" height="590"}

It will begin installing the recommended plugins.  This will take few minutes to complete.

![jenkins-getting-started : How to install Jenkins on Ubuntu](http://www.ethernetresearch.com/wp-content/uploads/2018/11/jenkins-getting-started.jpg){.aligncenter .wp-image-1239 .size-full width="1137" height="590"}

Next, it will prompt us to add new users to administer the Jenkins server.  We can skip this step and continue as an "admin" user using the password we have already provided to login. But, we'll configure a user named "ganesh" with some credentials, as shown below.

![Jenkins-Add-User : How to install Jenkins on Ubuntu](http://www.ethernetresearch.com/wp-content/uploads/2018/11/Jenkins-Add-User.jpg){.aligncenter .wp-image-1237 .size-full width="1024" height="706"}

After this is done.  We'll see the "Jenkins is Ready!" screen, as shown below.

![jenkins-ready : How to install Jenkins on Ubuntu](http://www.ethernetresearch.com/wp-content/uploads/2018/11/jenkins-ready.jpg){.aligncenter .wp-image-1240 .size-full width="1129" height="607"}

Click the "Start Using Jenkins" button to view the Jenkins Main Dashboard.

![Jenkinks-Dashboard : How to install Jenkins on Ubuntu](http://www.ethernetresearch.com/wp-content/uploads/2018/11/Jenkinks-Dashboard.jpg){.aligncenter .wp-image-1236 .size-full width="1024" height="595"}

At this point, Jenkins is successfully installed and configured.  You can start adding projects in Jenkins now.

 
