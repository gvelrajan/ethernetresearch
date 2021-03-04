---
title: How to Install SSL Certificate in WordPress VM
description: ""
author: Ganesh Velrajan
tags: [
    Linux VM, SSL Certificate, Wordpress, Cloud
]
date: 2018-07-19
categories: [
    GeekZone, Linux Networking
]
images: ["/images/linux/programming.jpg"]
---

In this article, we'll discuss how to install SSL Certificate in any Linux based VM, including the WordPress VM running in Google Cloud Platform(GCP)

It doesn't matter whether the VM is running in GCP or in your local server or anywhere.

The procedure is the same.

**Quick Trivia:**

The WordPress VM available in GCP Click to Deploy has the following artefacts in them.

    Debian 8

    Apache HTTP Server 2.4.10

    MySQL 5.5.58

    PHP 5.6.30

Anyway, that was just an FYI.  Let's move ahead with our demo.

## Installing SSL Certificate from Linux Bash Shell

\[ **Note**: You can skip steps \#1 and \#2 below for GCP WordPress VM because it already comes with Openssl and Apache2 installed in it. \]

First install Open SSL package in your Linux machine

### Step \#1

    sudo apt-get update

    sudo apt-get upgrade openssl

Next Install Apache.  Apache by default will run the HTTP service.  We'll configure it to run the HTTPS service.

### Step \#2

    sudo apt-get install apache2

Enable the SSL module in Apache.

    sudo a2enmod ssl

### Step \#3

Apache server comes with a default template for enabling SSL. Let's leverage it.

    sudo a2ensite default-ssl

The above command will create a "default-ssl.conf" in the /etc/apache2/sites-enabled/ folder.

    $ ls

    default-ssl.conf  wordpress.conf

### Step \#4

Reload the Apache server for these changes to take effect.

    $ sudo service apache2 reload

Why do you want your website to be SSL Certified or https enabled?

If your goal is to get "https" in your WordPress or any website so that it gives a good impression about your site to your users or for SEO reasons, then just stop buying SSL Certificates from those third-party private SSL Vendors who basically rob you of your money.  Instead get it for absolutely free, no obligation attached, using the below options.

### Step \#5

You Have Two Option:

a\) You can generate a self signed SSL Certificate in your machine locally

b\) Get a free SSL certificate from https://www.sslforfree.com/  which is a non-profit organization supporting this.  Follow the procedures in the website to get your SSL Certificate Files.

The site will give you an option to download the files in a zipped format.  Unzip them and bring it to your Linux VM and place them under /etc/apache2/ssl/ folder.   You'll have to create the ssl folder under /etc/apache2 before you can copy the files to this location.

After you copy the files.  Make the files not accessible by other users but only by the root. Use the following command.

    $ chmod 600 /etc/apache2/ssh/*

    $ ls -l /etc/apache2/ssl

    total 12

    -rw------- 1 root root 1646 Jul 11 11:09 ca_bundle.crt

    -rw------- 1 root root 2174 Jul 11 11:09 certificate.crt

    -rw------- 1 root root 1703 Jul 11 11:09 private.key

    $

### Step \#6

Now open the default-ssl.conf file that we generated earlier in step \#3.

    $ sudo vim /etc/apache2/sites-enabled/default-ssl.conf


    <IfModule mod_ssl.c>

            <VirtualHost _default_:443>

                    ServerAdmin webmaster@localhost

                    DocumentRoot /var/www/html

                    # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,

                    # error, crit, alert, emerg.

                    # It is also possible to configure the loglevel for particular

                    # modules, e.g.

                    #LogLevel info ssl:warn

                    ErrorLog ${APACHE_LOG_DIR}/error.log

                    CustomLog ${APACHE_LOG_DIR}/access.log combined

                    # For most configuration files from conf-available/, which are

                    # enabled or disabled at a global level, it is possible to

                    # include a line for only one particular virtual host. For example the

                    # following line enables the CGI configuration for this host only

                    # after it has been globally disabled with "a2disconf".

                    #Include conf-available/serve-cgi-bin.conf

                    #   SSL Engine Switch:

                    #   Enable/Disable SSL for this virtual host.

                    SSLEngine on

                    #   A self-signed (snakeoil) certificate can be created by installing

                    #   the ssl-cert package. See

                    #   /usr/share/doc/apache2/README.Debian.gz for more info.

                    #   If both key and certificate are stored in the same file, only the

                    #   SSLCertificateFile directive is needed.

                    SSLCertificateFile /etc/apache2/ssl/certificate.crt

                    SSLCertificateKeyFile /etc/apache2/ssl/private.key

    ...

    :wq

    $

Save the file and exit vim.

### 

### Step \#7 (Optional)

If you want you can rename the default-ssl.conf file to wordpress-ssl.conf file.  Already the folder would have a wordpress.conf file for the non-secure "http" version of the Apache instance.  This is not a mandatory step, but optional.

    $ sudo mv default-ssl.conf wordpress-ssl.conf

    $ ls /etc/apache2/sites-enabled

    wordpress.conf  wordpress-ssl.conf

### 

### Step \#8

Restart Apache so that the changes made to the config file will take effect.

    $ sudo service apache2 reload

 

### Step \#9

Verify that the HTTPS service runs on port 443.

    $ openssl s_client -connect 'your-VM-IP-Address-Here:443'

You'll see some output like this...

    -----BEGIN CERTIFICATE-----

    ...

    ...

    ...


    -----END CERTIFICATE-----

    ...

    ...

    ---

    No client certificate CA names sent

    ---

    SSL handshake has read 2257 bytes and written 415 bytes

    ---

    ...

    ...


    ---

    ^c

    read:errno=0

    $

If you see the above output then your SSL Certificate is installed and Apache HTTPS service is working just fine.  You can press CTRL+C and exit.

You can go to your web browser and make it point to the "https://your-website-name-here" and press enter.  It will show your website.  More importantly, it will show the lock symbol next the URL, meaning it is a secure connection.

We are done with successful installation and configuration on the Apache server side is done.

## Changes Required in WordPress:

Create a new or edit an already existing ".htaccess" file under /var/www/html/  
This is the root folder of your website.

Add the following content to the ".htaccess" file.

### Step \#10

You can do this by adding the following code in your .htaccess file:

    <IfModule mod_rewrite.c>

    RewriteEngine On

    RewriteCond %{SERVER_PORT} 80

    RewriteRule ^(.*)$ https://www.yoursite.com/$1 [R,L]

    </IfModule>

Replace 'www.yoursite.com' in the above configuration file with your site URL.

### Step \#11

Also edit wp-config.php file in the same root directory of your website and add the following line as shown below.

    define('WP_DEBUG', false);

    define('FORCE_SSL_ADMIN', true);

    /* That's all, stop editing! Happy blogging. */

    ...

### Step \#12

Finally, login to your WordPress admin console and go to the settings page.  Edit the URL of your website.  Change "http" to "https" as shown below.

Now if you point any web-browser to the "http" version of your website, it will redirect automatically to the "https" version of your website.

That's all folks !
