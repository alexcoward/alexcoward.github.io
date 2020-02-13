---
layout: post
title:  "Jenkins Disk Space Management"
date:   2020-02-13 09:48:45 -0700
---

## Managing Disk Space With Jenkins

<!--break-->

I love using Jenkins as a CI/CD platform, but one major problem with Jenkins is that unless you have defined a disk management
strategy from the onset, you will eventually run out of drive space on your Jenkins instance. Then you will be forced to take 
time to correct your error.

Below I will show you how to set a discard policy on Jenkins master so that Jenkins takes care of deleteing old artifacts and builds 
for you! In my example I will discard old artifacts every 365 days, and only keep 365 days worth of artifacts in each job folder.


### How can We Set The Discard Policy

1: First, we must navigate to the script console in Jenkins

{% highlight ruby %}

Log into Jenkins
Click on Manage Jenkins
Scroll down and click on the Script Console (The script console is where we can freestyle code to run on Jenkins master)


{% endhighlight %}

2: Now we need to create our Groovy script

{% highlight ruby %}

// NOTES:
// To only list the jobs which would be changed, not actually modify any data
dryRun = true
// If not -1, history is only kept up to this day.
daysToKeep = 365
// If not -1, only this number of build logs are kept.
numToKeep = -1
// If not -1 nor null, artifacts are only kept up to this day.
artifactDaysToKeep = 365
// If not -1 nor null, only this number of builds have their artifacts kept.
artifactNumToKeep = -1

Jenkins.instanceOrNull.allItems(hudson.model.Job).each { job ->
    if (job.isBuildable() && job.supportsLogRotator() && job.getProperty(jenkins.model.BuildDiscarderProperty) == null) {
        println "Processing \"${job.fullDisplayName}\""
        if (!"true".equals(dryRun)) {
            // adding a property implicitly saves so no explicit one
            try {
                job.setBuildDiscarder(new hudson.tasks.LogRotator ( daysToKeep, numToKeep, artifactDaysToKeep, artifactNumToKeep))
                println "${job.fullName} is updated"
            } catch (Exception e) {
                // Some implementation like for example the hudson.matrix.MatrixConfiguration supports a LogRotator but not setting it
                println "[WARNING] Failed to update ${job.fullName} of type ${job.class} : ${e}"
            }
        }
    }
}
return;


{% endhighlight %}

3: Now run the Groovy script in the script console, note that since we set the boolean dryRun to true, you will only see a list of the jobs whose
settings would be changed. No changes have been made. I would recommend you do a dryrun first to make sure all your jobs are visible. Then
you can even test it out on one job before applying it to Jenkins master.

The script below is an example of how you could change the discard policy of just one job.

{% highlight ruby %}

// NOTES:
// To only list the jobs which would be changed, not actually modify any data
dryRun = false
// If not -1, history is only kept up to this day.
daysToKeep = 365
// If not -1, only this number of build logs are kept.
numToKeep = -1
// If not -1 nor null, artifacts are only kept up to this day.
artifactDaysToKeep = 365
// If not -1 nor null, only this number of builds have their artifacts kept.
artifactNumToKeep = -1
// Job name to apply policy to
mName = "MyRepo/MyApp/Master"

Jenkins.instanceOrNull.allItems(hudson.model.Job).each { job ->
    if (job.fullName == mName && job.isBuildable() && job.supportsLogRotator() && job.getProperty(jenkins.model.BuildDiscarderProperty) == null) {
        println "Processing \"${job.fullDisplayName}\""
        if (!"true".equals(dryRun)) {
            // adding a property implicitly saves so no explicit one
            try {
                job.setBuildDiscarder(new hudson.tasks.LogRotator ( daysToKeep, numToKeep, artifactDaysToKeep, artifactNumToKeep))
                println "${job.fullName} is updated"
            } catch (Exception e) {
                // Some implementation like for example the hudson.matrix.MatrixConfiguration supports a LogRotator but not setting it
                println "[WARNING] Failed to update ${job.fullName} of type ${job.class} : ${e}"
            }
        }
    }
}
return;
		
{% endhighlight %}
