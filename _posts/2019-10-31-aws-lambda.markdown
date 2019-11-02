---
layout: post
title:  "AWS Lambda"
date:   2019-10-31 18:00:30 -0700
---

## AWS Lambda!

<!--break-->

Lambda in AWS is a convienant way to run an application or code without having to pay for a *server* to host your application/code.

According to AWS, "With AWS Lambda, you are charged for every 100ms your code executes and the number of times your code is triggered. 
You don't pay anything when your code isn't running." So basically you pretty much want to try and host fast and efficent applications...

We can also trigger our lambda function from cloudwatch, a nice way to automate things. For this example below we will create a function 
that starts and stops only our EC2 instances that are tagged properly. We can set up a cloudwatch event to trigger the start/stop as well.

* For this example I will be using Terraform to deploy the Lambda function

### Lambda Function

Note: Some steps are omited

1: We can write our functions using Python 2.7 (Yes its old...)

{% highlight ruby %}

import boto3
import logging

#setup simple logging for INFO
logger = logging.getLogger()
logger.setLevel(logging.INFO)

#define the connection
ec2 = boto3.resource('ec2','us-west-2')

def lambda_handler(event, context):
    # Use the filter() method of the instances collection to retrieve
    # all running EC2 instances.
    filters = [{
            'Name': 'tag:AutoOnOff', ##<------ This is our key value pair that MUST exist on our EC2 instances 
            'Values': ['True']
        },
        {
            'Name': 'instance-state-name', 
            'Values': ['stopped']
        }
    ]
    
    #filter the instances
    instances = ec2.instances.filter(Filters=filters)

    #locate all stopped instances
    StoppedInstances = [instance.id for instance in instances]
    
    #print the instances for logging purposes
    #print StoppedInstances 
    
    #make sure there are actually instances to start. 
    if len(StoppedInstances) > 0: ##<------ We need to not try and start instances that are not stopped
        #perform the startup
        startingUp = ec2.instances.filter(InstanceIds=StoppedInstances).start()
        print startingUp
    else:
        print "No Instances To Start!"
	  
{% endhighlight %}


{% highlight ruby %}

import boto3
import logging

#setup simple logging for INFO
logger = logging.getLogger()
logger.setLevel(logging.INFO)

#define the connection
ec2 = boto3.resource('ec2','us-west-2')

def lambda_handler(event, context):
    # Use the filter() method of the instances collection to retrieve
    # all running EC2 instances.
    filters = [{
            'Name': 'tag:AutoOnOff',
            'Values': ['True']
        },
        {
            'Name': 'instance-state-name', 
            'Values': ['running']
        }
    ]
    
    #filter the instances
    instances = ec2.instances.filter(Filters=filters)

    #locate all running instances
    RunningInstances = [instance.id for instance in instances]
    
    #print the instances for logging purposes
    #print RunningInstances 
    
    #make sure there are actually instances to shut down. 
    if len(RunningInstances) > 0:
        #perform the shutdown
        shuttingDown = ec2.instances.filter(InstanceIds=RunningInstances).stop()
        print shuttingDown
    else:
        print "No Instances To Stop!"
	  
{% endhighlight %}

2: In our .tf file we can use the aws_lambda_function functionality to declare our function

{% highlight ruby %}

# Lambda defined that runs the Python code with the specified IAM role
resource "aws_lambda_function" "ec2_start_scheduler_lambda" {
filename = "start_instances.zip" ##<------ Notice that we have to zip our .py file
function_name = "start_instances"
role = "${aws_iam_role.lambdarole.arn}"
handler = "start_instances.lambda_handler"
runtime = "python2.7"
timeout = 300
source_code_hash = "${filebase64sha256("./start_instances.zip")}"
}

{% endhighlight %}

3: We can schedule our lambda function to run in cloudwatch by using the Terraform aws_cloudwatch_event_rule functionality 

{% highlight ruby %}

### Cloudwatch Events ###
# Event rule: Runs at 8pm during working days
resource "aws_cloudwatch_event_rule" "start_instances_event_rule" {
  name = "start_instances_event_rule"
  description = "Starts stopped EC2 instances"
  schedule_expression = "cron(0 8 ? * MON-FRI *)" ##<--------- like chrontab we can schedule our event to run a specific times
  depends_on = ["aws_lambda_function.ec2_start_scheduler_lambda"]
}

{% endhighlight %}

### Permissions

Below is an example of the persmissions needed to schedule our cloudwatch event

1: We can use aws_lambda_permission in Terraform to give us permission to add the scheduler

{% highlight ruby %}

# AWS Lambda Permissions: Allow CloudWatch to execute the Lambda Functions
resource "aws_lambda_permission" "allow_cloudwatch_to_call_start_scheduler" {
  statement_id = "AllowExecutionFromCloudWatch"
  action = "lambda:InvokeFunction"
  function_name = "${aws_lambda_function.ec2_start_scheduler_lambda.function_name}"
  principal = "events.amazonaws.com"
  source_arn = "${aws_cloudwatch_event_rule.start_instances_event_rule.arn}"
}
 
{% endhighlight %}

2: We need to have a IAM role and policy to attach to our IAM role as well.

{% highlight ruby %}

# create Lambda role start/stop
resource "aws_iam_role" "lambdarole" {
  name = "project1-lambda"

  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Effect": "Allow",
      "Sid": ""
    }
  ]
}
EOF

  tags = {
    tag-key = "project-project1"
  }
}

# create IAM policy for lamdba
resource "aws_iam_policy" "project1lambdapolicy" {
  name        = "project1-lambda-policy"
  description = "Allows Lambda functions to call AWS services on your behalf."

  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*", ##<----- describe all our instances
        "ec2:Start*",    ##<----- Start our instances
        "ec2:Stop*"      ##<----- Stop our instances
      ],
      "Resource": "*" ##<----- All instances
    }
  ]
}
EOF
}

{% endhighlight %}

Thats pretty much it... After that we can use the console to trigger our Lambda function to test it!
Then it will stop our instances Monday - Friday at 8PM and start our instances on Monday - Friday at 8AM
