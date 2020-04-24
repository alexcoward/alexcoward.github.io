---
layout: post
title:  "A Simple Mac Signing Script"
date:   2020-04-24 08:40:50 -0700
---

## A Simple Mac App Signing Script

<!--break-->

If you are wanting to sign an app you built for MacOS you might run into an issue where your apps fails the notarization process if you did not nest your projects code properly. This is especially
true if you have included nested executables. You should always follow Apple's guidelines for nested code, at risk of having your project rejected by Apple's
notarization tool.

[https://developer.apple.com/library/archive/documentation/Security/Conceptual/CodeSigningGuide/Procedures/Procedures.html#//apple_ref/doc/uid/TP40005929-CH4-TNTAG201](https://developer.apple.com/library/archive/documentation/Security/Conceptual/CodeSigningGuide/Procedures/Procedures.html#//apple_ref/doc/uid/TP40005929-CH4-TNTAG201)

In this blog post I detail how you can sign nested executables via command line interface.

### Some Details First:
* It is not recommended that you sign third party executables unless you fully trust the executable
* By digitally signing your application you are essentially asserting that you are a point of trust in the distribution of the code
* You will need a developer code signing identity from Apple
* You will need Xcode command line tools installed

### My Experience

Signing apps via command line on a Mac is very easy, but its also very easy to not complete the task correctly. Signing an app is only
one step in the process of making your application available to users. The app deployment process would look something like this;

* Signing <-----
* Notarization
* Stapling
* Deploy

We will use the following flags for the codesign command

* -s  Sign the code at the path(s) given using this identity
* --timestamp During signing, requests that a timestamp authority server be contacted to authenticate the time of signing. The server contacted is given by the URL value. If this option is given without a value, a default server provided by Apple is used
* --options=runtime  Forces the signed code's hard flag to be set when the code begins execution. The hard flag is a hint to the system that the code prefers to be denied access to resources if gaining such access would invalidate its identity

The bad part about signing is you may never know if its done properly until you submit your code to Apple for notarization....

## How do I get started?


1: For this task we will use a bash loop to loop through all files in your project and digitally sign each file manually
* Note use this option if your project contains no sub-folders

{% highlight ruby %}

#!/bin/bash

DIR=~/path/to/.app/*

for file in $DIR; do
	echo "Signing $file"
	codesign --timestamp --options=runtime -s <Identity In Keychain> $file
done

{% endhighlight %}

OR if your files contain sub-folders you can do something like this

{% highlight ruby %}

#!/bin/bash

DIR=~/path/to/.app

find $DIR -type f | while read filename; do echo "Signing $file"; codesign --timestamp --options=runtime -s <Identity In Keychain> $file; done

{% endhighlight %}

2: Verify the signature

{% highlight ruby %}

codesign --verify --deep -vvvv /path/to/.app
spctl -a -vvvv /path/to/.app

{% endhighlight %}
