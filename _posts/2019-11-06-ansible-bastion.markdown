---
layout: post
title:  "Ansible and Bastion Hosts"
date:   2019-11-06 15:00:30 -0700
---

## Ansible and Bastion Hosts!

<!--break-->

For Project1 we were required to be able to run our Ansible playbooks in place locally and deploy to both a bastion server (public)
and two EC2 instances in a private subnet.

Because we had decided to use Instance connect this posed a bit of a problem as we had no ssh keys to ssh into our bastion or servers
for that matter!

The following is an example of how we solved the problem and future improvements that we can make to the make our program more user
friendly.


### Write A Bash Function to do all the work

To solve our problem we must first write a script that will ssh into the bastion and then deploy the Ansible
playbook to both remote servers. But wait... we dont use ssh! Well, lets invoke the AWS CLI API instead
as there is a function we can use called *send-ssh-public-key

1: We can use the function below to send a ssh public key to our remote webservers

{% highlight ruby %}

aws ec2-instance-connect send-ssh-public-key --region us-west-2 --instance-id " " --instance-os-user ubuntu \
--availability-zone " " --ssh-public-key file://my_rsa_key.pub

{% endhighlight %}

2: We will need to generate a RSA keypair as well to push to the server, lets call is my_rsa_key

{% highlight ruby %}

ssh-keygen -q -t rsa -f my_rsa_key

{% endhighlight %}

3: Once we have the RSA public key on the remote server we only have 60 seconds to ssh into the server and execute our playbook!

{% highlight ruby %}

ansible-playbook -i path/to/hosts path/to/.yml playbook

{% endhighlight %}

3: So, remember - our bastion is where we want to execute the script locally, so all our config files must exist locally and additionally,
 we want to setup the bastion as well. We can do this with one playbook if we divide our deployment up between hosts, for example our hosts
 file might look something like this

{% highlight ruby %}

# EC2 Webservers hosting artifactory
[webservers]
server1 ansible_host=IP.ADDRESS ansible_ssh_private_key_file=/home/ubuntu/my_rsa_key ansible_ssh_user=ubuntu
server2 ansible_host=IP.ADDRESS ansible_ssh_private_key_file=/home/ubuntu/my_rsa_key ansible_ssh_user=ubuntu

# Load balancer
[loadbalancer]

# Bastion Server
[bastion]
127.0.0.1 // localhost!

{% endhighlight %}

Then, when we invole ansible we split our tasks to be executed amongst the hosts! 

{% highlight ruby %}

hosts: webservers

// do stuff to the webservers remotely

hosts: bastion

// do stuff to the bastion locally

{% endhighlight %}

4: Now, we need to find a way to execute a script that exists locally, on a remote server using mssh. We can do that easily by
redirecting the content of our script to standard input to be read by the bash shell on the remote bastion.

{% highlight ruby %}

mssh ubuntu@[aws-EC2-id] "bash -s" < /path/to/script

Or even possibly

mssh ubuntu@[aws-EC2-id] < /path/to/script
 
{% endhighlight %}

### Our full script to invoke an ansible playbook on a remote Ubuntu Server

{% highlight ruby %}

#!/bin/bash

#
# Execute script from desktop
#
#Enter The AWS instance id of Server1 [i-XXXX]
ID1=""
#Enter The AWS instance DNS of Server1 [EC2.PUBLIC.DNS]
DNS1=""
#Enter The AWS availability zone of Server1 [us-west-2c]
AZ1=""
#Enter The AWS instance id of Server2 [i-XXXX]
ID2=""
#Enter The AWS instance DNS of Server2 [EC2.PUBLIC.DNS]
DNS2=""
#Enter The AWS availability zone of Server2 [us-west-2c]
AZ2="us-west-2c"
#AWS CLI variables
DEFAULT_ACCESS_KEY=""
DEFAULT_SECRET_KEY=""
#setup AWS CLI
aws configure set aws_access_key_id "$DEFAULT_ACCESS_KEY"
aws configure set aws_secret_access_key "$DEFAULT_SECRET_KEY"
aws configure set default.region "us-west-2"
#disable host key checking
export ANSIBLE_HOST_KEY_CHECKING=False
#clone repo
git clone https://github.com/alexcoward/Project1Infrastructure.git
#generate rsa keypair
yes y |ssh-keygen -q -t rsa -f my_rsa_key -N '' >/dev/null
#send ssh keys and execute playbook
aws ec2-instance-connect send-ssh-public-key --region us-west-2 --instance-id "$ID1" --instance-os-user ubuntu \
--availability-zone "$AZ1" --ssh-public-key file://my_rsa_key.pub && aws ec2-instance-connect send-ssh-public-key \
--region us-west-2 --instance-id "$ID2" --instance-os-user ubuntu --availability-zone "$AZ2" --ssh-public-key file://my_rsa_key.pub \
&& ansible-playbook -i /home/ubuntu/Project1Infrastructure/ansible/hosts /home/ubuntu/Project1Infrastructure/ansible/setup_all.yml
#cleanup
rm -rf Project1Infrastructure/
rm -f my_rsa_key
rm -r my_rsa_key.pub
#exit the shell gracefully
exit 0

{% endhighlight %}

### Improvements

1: If you noticed we sure do have to input a bunch of variables to make this script work! Technically, we could grab these variables from a file that is
written at run time when we build our infrastructure though!  

2: We could also write our hosts file dynamically by saving the IP addresses to a local file and reference that instead of having to modify the hosts
file each time we build new infrastructure!  