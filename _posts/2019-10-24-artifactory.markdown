---
layout: post
title:  "Artifactory"
date:   2019-10-24 17:00:30 -0700
---

## Artifactory!

<!--break-->

This week I learned about a nifty piece of software called Artifactory. Artifactory is basically a binary repository for, in my case, storing JAVA
binaries (artifacts) like .war's and .jar's. We can use Artifactory to host these reusable libraries (binaries) internally. For example, in theory during the build process
(for an Java Android app) we could use Gradle to go grab the binary from our Artifactory instance and use it inside our Android project.

Artifactory comes bundled with the Tomcat servlet so it technically can be used as a stand alone installation to serve binaries.

### We can setup Artifactory on Ubuntu Server 14 or 16

Note: Some steps are omited

Requirements:

Artifactory has some hefty requirements that we MUST satisfy
* 64bitOS
* 4Cores
* 4GB Heap (RAM) for JVM
* Fast disk with free space that is at least 3 times the total size of stored artifacts

My Install (AWS EC2):
* Ubuntu Server 16
* 8GB RAM
* 16GB Elastic Block Storage
* Running Apache2 for serving the frontend -> then handing off to Tomcat to serve the request back
* Use Ansible for automation! Note: code below is to be used in an Ansible playbook.

1: First we must add the Java development kit (JDK). Artifactory must run with JDK 8 (JDK 8 update 45 and above preferred)
or JDK 11 (from Artifactory version 6.10).

2: We must also add the JAVA_HOME environmental variable to our environment and additionally our /tmp/project1/config/default file

{% highlight ruby %}

- name: insert JAVA_HOME in .bashrc 
    lineinfile:
      path: /home/ubuntu/.bashrc
      line: "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64"

- name: insert PATH in .bashrc
    lineinfile:
      path: /home/ubuntu/.bashrc
      line: "export PATH=$PATH:$JAVA_HOME/bin"
	  
{% endhighlight %}

3: We can grab Artifactory in one nice .zip file, we will want to install it under /opt as it runs on port 8081

{% highlight ruby %}

- name: get artifactory .zip
    shell: sudo curl -L "https://jfrog.bintray.com/artifactory/jfrog-artifactory-oss-6.13.1.zip" -o jfrog-artifactory-oss-6.13.1.zip
    args:
      chdir: /opt
	  
- name: install artifactory
    shell: ./installService.sh
    args:
      chdir: /opt/artifactory-oss-6.13.1/bin

{% endhighlight %}

### Apache2 Setup

The trick here is maybe we dont want to have to enter the port# in our url. Well, turns out we can use Apache to proxy Artifactory via HTTP.

1: We need to modify our default 000-default-conf file as follows

{% highlight ruby %}

 <VirtualHost *:80>
        ServerAdmin alexander.coward.17@my.csun.edu
        DocumentRoot /var/www/html
        ServerName teamaerialcsun.gq
        ServerAlias www.teamaerialcsun.gq
        ErrorLog ${APACHE_LOG_DIR}/error.log
        ProxyPreserveHost on
        ProxyPass /artifactory http://localhost:8081/artifactory
        ProxyPassReverse /artifactory http://localhost/artifactory
        # Rewrite requests for / to /artifactory
        RewriteEngine on
        RewriteCond %{REQUEST_URI} ^/$
        RewriteRule (.*) /artifactory [R=301]]
</VirtualHost>
 
{% endhighlight %}

2: Next we can restart apache2

{% highlight ruby %}

- name: restart apache2
    service:
      name: apache2
      state: restarted

{% endhighlight %}

3: After that we can hit our URL using document root (/) on port 80 and we should see the UI for our Artifactory.

<br><br>

## Next Steps

You can read more about installing Artifactory below!  
[https://www.jfrog.com/confluence/display/RTF/Configuring+a+Reverse+Proxy](https://www.jfrog.com/confluence/display/RTF/Configuring+a+Reverse+Proxy)  
[https://www.jfrog.com/confluence/display/RTF/Working+with+Gradle](https://www.jfrog.com/confluence/display/RTF/Working+with+Gradle)  
[https://www.jfrog.com/confluence/display/RTF/System+Requirements](https://www.jfrog.com/confluence/display/RTF/System+Requirements)  

