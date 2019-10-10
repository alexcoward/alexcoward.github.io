---
layout: post
title:  "Project 0 Continued!"
date:   2019-10-10 14:55:30 -0700
---

## Finishing Up Project 0 This Week.

<!--break-->

So my group of two finished up project0 this week and we closed out the remaining tasks successfully
I will only briefly list items as you may visit our repo/website/wiki below to see the action items for yourself!

### How's the project coming along?

Our GitHub repo
[https://github.com/alexcoward/Project0](https://github.com/alexcoward/Project0)

Our GitHub project (two week sprint)
[https://github.com/alexcoward/Project0/projects](https://github.com/alexcoward/Project0/projects)

Our website
[https://teamaerialcsun.gq/](https://teamaerialcsun.gq/)

### Tasks Completed this week

1: We finished project0 documentation via our GitHub Wiki
[https://github.com/alexcoward/Project0/wiki](https://github.com/alexcoward/Project0/wiki)

2: Kindra finished setting up our VPC (public and private subnets) and set our EC2 in the public subnet
for simplicity, vs. having to attach a load balancer at the moment.

3: We setup slides for our presentation
[https://docs.google.com/presentation/d/1h7cWw4rx9umf6pcoB1iMuNshAQwVLrJPPi2AObfcMXA](https://docs.google.com/presentation/d/1h7cWw4rx9umf6pcoB1iMuNshAQwVLrJPPi2AObfcMXA)

<br><br>

## Some helpful commands I learned

1: You can make Ansible interactive (similar to `read -p "m_text" variable_name` in bash!) by doing the following. Note that the 
users input can be retrieved by accessing the `name:` property.

{% highlight ruby %}

 vars_prompt:
    - name: mrepo
      prompt: "What is your GitHub repo name?"
      private: no

    - name: musername
      prompt: "What is your GitHub username?"
      private: no

    - name: mpassword
      prompt: "What is your Github password?"

    - name: motp
      prompt: "Enter six digit one time password"

{% endhighlight %}
