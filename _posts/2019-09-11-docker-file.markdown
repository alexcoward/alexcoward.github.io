---
layout: post
title:  "Dockerfile Lab!"
date:   2019-09-11 16:13:49 -0700
---

## Completed lab1 and created my first Dockerfile.

<!--break-->

For this assignment I chose to create a 100% automated dockerfile that would essentially
install all binaries necessary to run a GitHub hosted CSUN web app called Affinity.

### I broke this project into multiple steps:

Step 1: Install core binaries

{% highlight ruby %}

RUN apt-get update -y \
	&& apt-get upgrade -y && apt-get install -y \
	vim \
	git \
	curl \
	zip \
	apache2 \
	mysql-client
	
{% endhighlight %}

Step 2: Install project dependencies. One problem I ran into here was that php uses 
an interactive install (meaning it prompts the user for data necessary to set the 
time zones, etc. located in the tzinfo package) to overcome this I run the install
in non-interactive mode.

{% highlight ruby %}

RUN DEBIAN_FRONTEND=noninteractive apt-get install -y \
	php \
	libapache2-mod-php \
	php-mysql \
	php7.2-cli \
	php7.2-curl \
	php7.2-gd \
	php7.2-mbstring \
	php7.2-mysql \
	php7.2-xml
	
{% endhighlight %}

Step 3: Clone the affinity app repo and change ownership of the folder to www-data. Also I chose to edit in place 
using sed -i the apache2 config file (as the Documentroot needs the be changed from /var/www/html to /var/www/html/public.

{% highlight ruby %}

RUN cd /var/www/html && git clone https://github.com/csuntechlab/affinity.git \
	&& chown -hR www-data:www-data affinity/ && ln -s /var/www/html/affinity/public public \
	&& sed -i 's!/var/www/html!/var/www/html/public!g' /etc/apache2/sites-available/000-default.conf
	
{% endhighlight %}

Step 4: Edit the apache2 conf. Here, much like above I chose to use the sed stream editor to modify two area of the
Apache2 conf file (change require all denied to require all granted & /var/www to /var/www/html/public. Additionally,
I found it necessary to add the ServerName into the conf file because I would receive the following error when running
apache2 "Could not reliably determine the server's fully qualified domain name”.

{% highlight ruby %}

RUN sed -i '/<Directory \/>/,/<\/Directory>/ s/Require all denied/Require all granted/' /etc/apache2/apache2.conf \
	&& sed -i '/<Directory \/var\/www\/>/,/<\/Directory>/ s/Directory \/var\/www\//Directory \/var\/www\/html\/public/' /etc/apache2/apache2.conf \
	&& echo "ServerName localhost" >> /etc/apache2/apache2.conf
	
{% endhighlight %}

Step 5: Install composer

{% highlight ruby %}

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin \
	--filename=composer && cd /var/www/html/affinity && composer install
	
{% endhighlight %}

Step 6: Run Apache in the foreground of the container. This allows us to not have to manually enter into the container 
and start apache2. Instead the command CMD will be executed and run in the foreground of the container.

{% highlight ruby %}

CMD ["/usr/sbin/apachectl", "-D", "FOREGROUND"]

{% endhighlight %}

<br><br>

## I also learned some very nice commands for future use in the docker environment.

{% highlight ruby %}

docker build -t <image name> .  // To build an image from a Dockerfile
docker run -dt -p 8080:80 --name <container name> <image name> // To run the container as a background (non-blocking) process mapped to port 8080
docker run -it -p 8080:80 --name <container name> <image name> // To run the container as a foreground (blocking) process mapped to port 8080

{% endhighlight %}
