---
layout: post
title:  "GitHub APIv3"
date:   2020-03-13 07:50:50 -0700
---

## Brief Into To GitHub APIv3

<!--break-->

GitHub has made it really easy for developers to integrate GitHub actions into their applications or scripts via a restful public API called
GitHub APIv3.

In this post I will show some of the basic things we can do with the GitHub API via scripting.

### Using The API

1: For a non organizational user we can see the users attributes using curl (note that: you will be prompted for a password)


{% highlight ruby %}

curl -u username https://api.github.com/user

{% endhighlight %}


2: For an organization that supports SAML SSO, we can create a SAML SSO personal access token to access a repo via curl

{% highlight ruby %}

curl -v -H "Authorization: token TOKEN" https://api.github.com/repos/{org}/{repo-name}

{% endhighlight %}

3: We can list all repos owned by a user (note that: you will be prompted for a password)

{% highlight ruby %}

 curl -i -u username  https://api.github.com/users/username/repos?type=all

{% endhighlight %}

4: Many API methods take optional parameters. For GET requests, any parameters not specified as a segment in the path can be passed as an HTTP query string parameter

{% highlight ruby %}

// In this example, the 'vmg' and 'redcarpet' values are provided for the :owner and :repo parameters in the path while :state is passed in the query string.

curl -i "https://api.github.com/repos/vmg/redcarpet/issues?state=closed"

{% endhighlight %}

5: To download a file from a public repo

{% highlight ruby %}

curl -O -H 'Accept: application/vnd.github.v3.raw' https://api.github.com/repos/alexcoward/Project0/contents/README.md

{% endhighlight %}
