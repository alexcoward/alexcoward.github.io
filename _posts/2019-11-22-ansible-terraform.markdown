---
layout: post
title:  "Ansible And Terraform"
date:   2019-11-22 09:10:38 -0700
---

## Ansible And Terraform!

<!--break-->

I recently learned that Ansible can be used to execute a Terraform plan. This seems like a very nice feature,
as we might want to automate our infrastructure deployment through Terraform and then use Ansible to provision
the infrastructure. 

### How does the terraform_module work?

1: First, we need to create our Terraform files in a folder. Our folder should contain
* main.tf  
* variables.tf  
* outputs.tf  

2: We can execute our Terraform deployment from Ansible by using the Ansible module: terraform_module

{% highlight ruby %}

- hosts: localhost
  connection: local
  gather_facts: no
  vars:
     project_dir : "$home/project1"
  tasks:
  
  -name: terraform apply
    terraform:
      project_path: "{{ "{{ project_dir" }}}}" // The path to the root of the Terraform directory with the vars.tf/main.tf/etc to use.
      state: present

{% endhighlight %}

3: We can also execute a Terraform apply and reference our existing Terraform backend like this

{% highlight ruby %}

- hosts: localhost
  connection: local
  gather_facts: no
  vars:
     project_dir : "$home/project1"
  tasks:
  
  -name: terraform apply with backend
    terraform:
      project_path:  "{{ "{{ project_dir" }}}}" 
      state: present
	  force_init: true // If a state file cant be found this will force a `terraform init`.
      backend_config: // A group of key-values to provide at init stage to the -backend-config parameter.
        region: "us-west-2"
        bucket: "some-bucket"
        key: "random.tfstate"

{% endhighlight %}

3: Then to call our ansible playbook we execute

{% highlight ruby %}

ansible-playbook project1.yml

{% endhighlight %}

In the above example we use the Ansible module `terraform_module` to call Terraform and thus initiate a deployment of some infrastructure to
our AWS account.
