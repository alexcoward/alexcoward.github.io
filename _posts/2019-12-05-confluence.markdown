---
layout: post
title:  "Confluence"
date:   2019-12-05 09:10:45 -0700
---

## Confluence!

<!--break-->

Confluence is a collaboration type application (think Wiki) that allows teams to better share knowledge 
related to development projects. Most companies use confluence to document how a development team operates and
additionally many use Confluence as a "central" repository to house project documentation related to projects that
the team has delivered in the past.

To put it simply, with Confluence, content is created and organized using spaces, pages, and blogs.

Since Confluence is an Atlassian product so it also integrates nicely with JIRA. For example when you bundle 
Confluence with JIRA (they should be installed on different servers of course) you may:
* Display an entire list of JIRA issues within a Confluence page. Issues can be filtered and sorted however you like
* View JIRA issue due dates, milestones, and sprints alongside your people and team events calendars
* Get a single, dynamic view of all the sprints, epics, and issues related to your requirements at the top of the page ("JIRA links" button)
* Link to a confluence page from your JIRA dashboard
* Link to a confluence page from a JIRA issue

### How might we deploy Confluence?

1: First, we need to understand the use case and provision our servers accordingly
* Seperate application server for Confluence
* Seperate database server (PostgreSQL is recommended by Atlassian)
Each Server:
* 64bit OS 
* CPU: Quad core 2GHz+ CPU
* RAM: 6GB
* Minimum database space: 10GB

2: We can install the Confluence server bundle by downloading the bundled installer at

[Confluence Download Link](https://www.atlassian.com/software/confluence/download)

After downloading we would execute the following commands:

{% highlight ruby %}
//Make the installer executable
chmod a+x atlassian-confluence-7.1.1-x64.bin

//Run the installer locally on server as sudo to install as a service (easy start and stop)
sudo ./atlassian-confluence-7.1.1-x64.bin

{% endhighlight %}

3: Next we can follow the setup prompts to install Confluence

* Install type – choose option 2 (custom) for the most control. 
* Destination directory – this is where Confluence will be installed.
* Home directory – this is where Confluence data like logs, search indexes and files will be stored.
* TCP ports – these are the HTTP connector port and control port Confluence will run on. Stick with the default unless you're running another application on the same port.
* Install as service – this option is only available if you ran the installer as sudo. 

3: Once installation is complete head to http://localhost:8090/ in your browser to begin the setup process. 

* Choose installation type
* Enter license
* Connect to the external database
* Populate site with content
* Manage users
* Create Admin account

4: Setup the database server
* With PostgreSQL Confluence currently supports v9.5 & v9.
* Ubuntu 18
{% highlight ruby %}

sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -

sudo apt-get update
sudo apt-get upgrade
sudo apt-get install postgresql-9.6
sudo apt install pgadmin4 pgadmin4-apache2 // If you want the pgadmin GUI

{% endhighlight %}

Or we can download the bundled installer which contains pgAdmin!
[PostgreSQL Bundle Download](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)

5: PostgreSQL runs on port :5432 by default
* Next you can create a database user
* Create confluence database (Character encoding must be set to utf8 encoding and Collation must also be set to utf8)
* Use the JDBC connection to connect to the Confluence server

6: Test your database connection using the test connection button on the database setup screen!
