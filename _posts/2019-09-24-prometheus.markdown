---
layout: post
title:  "Prometheus Lab!"
date:   2019-09-24 18:45:50 -0700
---

## Completed the Prometheus lab.

<!--break-->

This was my first experience using Prometheus or Node Exporter.

### Well, what exactly is Prometheus?

Prometheus is an open-source systems monitoring and alerting toolkit.

1: We can install Prometheus via wget:

{% highlight ruby %}

wget https://github.com/prometheus/prometheus/releases/download/v2.8.0/prometheus-2.8.0.linux-amd64.tar.gz 
	
{% endhighlight %}

2: We can run Prometheus by typing (by default it seems to run on port 9090):

{% highlight ruby %}

./prometheus --config.file=prometheus.yml
	
{% endhighlight %}



### And what is Node Exporter?

Node Exporter is a Prometheus exporter for hardware and OS metrics with pluggable metric collectors. 
It allows to measure various machine resources such as memory, disk and CPU utilization.

1: We can install Node Exporter via wget

{% highlight ruby %}

wget https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
	
{% endhighlight %}

2: We can run Node Exporter by typing (by deafault it seems to run on port 9100):

{% highlight ruby %}

./node-exporter 
	
{% endhighlight %}

<br><br>

## Some helpful commands I learned

1: To run multiple programs in parallel in the foreground:

{% highlight ruby %}

./prog1 & ./prog2

{% endhighlight %}

2: To commit a docker container (in order to create a new image based on an existing image):

{% highlight ruby %}

docker commit CONTAINER_ID CHOSEN_NAME

{% endhighlight %}

3: To build a new container based on commited image, do some port mapping and open up a bash terminal:

{% highlight ruby %}

docker run -it --name CONTAINER_NAME -p 9090:9090 -p 9100:9100 CHOSEN_NAME bash

{% endhighlight %}