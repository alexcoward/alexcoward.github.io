---
layout: post
title:  "Service Now"
date:   2020-04-03 10:50:50 -0700
---

## Service Now

<!--break-->

Service Now is a very popular application used by many large companies to manage IT service desks, employee and customer portals,
and additionally business processes which can involve complex work flows. The Service Now software is available as a SaaS (Software as a Service) subscription, 
or as an on-premise deployment. The model chosen (SaaS vs. on-premise) is usually dictated by the nature of the data to be stored in the system.

In this blog post I will give some resources for learning how to develop applications using Service Now.

### Developers Program 

Service Now offers a free developers program, which allows you to do many cool thing such as:

* Taking online training courses in which you learn about the platform
* The ability to access their dev cloud and spin up dev instances of Service Now to experiment with  
* The ability to access their community portal 
* Access to sample code
* The ability to share code with other developers (paid or free)
* Of course you can access documentation for all the releases

The developers portal is accessible at the following URL
[https://developer.servicenow.com/dev.do](https://developer.servicenow.com/dev.do)

### My Experience

This week I took some time to complete a few of their development exercises. I used a Personal Developer Instance (similar to an AWS EC2)
in order to complete the exercises. Some of the exercises were very simple, like creating and adding a field to a form. Some were
more complex, like creating a business application or creating workflows for an existing application. Overall I found the training 
courses useful and easy to follow along. The one thing I really liked was that their system in pretty much all-in. Meaning if
you buy into Service Now, everything software-wise you need is there to maintain and build your system. For example, when developing 
custom apps you dont have to use your own IDE (Integrated Development Environment)with version control. They actually have an IDE built into the software,
its called Studio IDE, you can indeed fork open source GutHub repo's, clone your own repo's, commit, push, etc. However, if you choose to use another IDE
apparently Microsoft Visual Studio Code (VS Code) has a plugin that supports Service.


## Some Things To Know

There are three versions of Service Now in which you may request for your Personal Developer Instance:

* Orlando (Newest)
* New York
* Madrid

The software itself is a absolute beast, it will take some time to get used to configuring and writing custom apps for sure!

Yes, your custom Service Now application can be made mobile compatible! There are multiple Service Now applications available 
for download for iOS and Android devices.

Yes, Service Now has a robust set of APIs. The Service Now Platform provides various REST APIs, which are active by default. 
These APIs provide the ability to interact with various ServiceNow functionality within your own custom application. 
Such functionality includes the ability to perform create, read, update, and delete (CRUD) operations on existing tables (Table API)

## How do I get started?

1: Sign up to be a part of the developer program here:

[https://developer.servicenow.com/dev.do](https://developer.servicenow.com/dev.do)


2: Follow these instruction in order to launch a personal developer instance:

[https://developer.servicenow.com/dev.do#!/guide/newyork/now-platform/pdi-guide/obtaining-a-pdi](https://developer.servicenow.com/dev.do#!/guide/newyork/now-platform/pdi-guide/obtaining-a-pdi)
