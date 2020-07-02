---
layout: post
title: Create an AWS S3 Bucket with Terraform
date: 2020-07-01 21:55
category: Cloud
author: Anand Rao
tags: [Cloud , AWS , Storage , terraform]
summary: A Simple walk through to a process of creating an S3 Bucket using Terrarorm
---
Amazon Simple Storage Service (Amazon S3) as defined on the AWS docs is an object storage service that offers industry-leading scalability, data availability, security, and performance. 

If you already have an AWS account ,  you can create the bucket or containers on this service via the AWS console and the `aws` cli. Both of these are more of an imperative model of creating this. In the blog I will show how to use the declarative model of creating the buckets using the Hashicorp terraform. 

#### 1. Create a working folder for the configurations. 
    {% highlight bash %}
    $ mkdir s3-playground
    {% endhighlight %}
#### 2. Create a file called providers.tf with contents as follows
    {% highlight tf %}
    provider "aws" {
      region = var.aws_region
      profile = var.aws_profile
    }
    
    terraform {
    }
    {% endhighlight %}

#### 3. Create a file called variables.tf with contents as follows
    {% highlight tf %}
        variable "aws_s3_bucket_name" {}
        variable "aws_region" {
            default = "us-west-2"
        }
        variable "aws_profile" {
            default = "default"
        }
        {% endhighlight %}
    
#### 4. Create a file called outputs.tf with contents as follows
    {% highlight tf %}
    output "terraform_state_bucket" {
      value = aws_s3_bucket.the_bucket.arn
    }
    
    output "bucket_domain_name" {
      value = aws_s3_bucket.the_bucket.bucket_domain_name 
    }
    
    output "bucket_regional_domain_name" {
      value = aws_s3_bucket.the_bucket.bucket_regional_domain_name 
    }
    
    output "hosted_zone_id" {
      value = aws_s3_bucket.the_bucket.hosted_zone_id
    }
    
    output "bucket_id" {
      value = aws_s3_bucket.the_bucket.id
    }
    
    output "region" {
      value = aws_s3_bucket.the_bucket.region
    }
    {% endhighlight %}

#### 5. Create a file called outputs.tf with contents as follows
    {% highlight tf %}
    aws_s3_bucket_name = "rao-ksm-chartmuseum"
    aws_region = "us-west-2"
    aws_profile = "default"
    {% endhighlight %}

#### 6. Execute the below commands 
    {% highlight bash %}
    $ terraform init
    $ terraform validate
    $ terraform plan
    $ terraform apply
    {% endhighlight %}
    
#### Sample output should look like this below:

    {% highlight bash %}
    $ terraform init ; terraform validate ; terraform plan  ; terraform apply 
    
    Initializing the backend...
    
    Initializing provider plugins...
    
    The following providers do not have any version constraints in configuration,
    so the latest version was installed.
    
    To prevent automatic upgrades to new major versions that may contain breaking
    changes, it is recommended to add version = "..." constraints to the
    corresponding provider blocks in configuration, with the constraint strings
    suggested below.
    
    * provider.aws: version = "~> 2.68"
    
    Terraform has been successfully initialized!
    
    You may now begin working with Terraform. Try running "terraform plan" to see
    any changes that are required for your infrastructure. All Terraform commands
    should now work.
    
    If you ever set or change modules or backend configuration for Terraform,
    rerun this command to reinitialize your working directory. If you forget, other
    commands will detect it and remind you to do so if necessary.
    Success! The configuration is valid.
    
    Refreshing Terraform state in-memory prior to plan...
    The refreshed state will be used to calculate this plan, but will not be
    persisted to local or remote state storage.
    
    aws_s3_bucket.the_bucket: Refreshing state... [id=rao-ksm-chartmuseum]
    
    ------------------------------------------------------------------------
    
    No changes. Infrastructure is up-to-date.
    
    This means that Terraform did not detect any differences between your
    configuration and real physical resources that exist. As a result, no
    actions need to be performed.
    aws_s3_bucket.the_bucket: Refreshing state... [id=rao-ksm-chartmuseum]
    
    Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
    
    Outputs:
    
    bucket_domain_name = rao-ksm-chartmuseum.s3.amazonaws.com
    bucket_id = rao-ksm-chartmuseum
    bucket_regional_domain_name = rao-ksm-chartmuseum.s3.us-west-2.amazonaws.com
    hosted_zone_id = Z3BJ6K6RIION7M
    region = us-west-2
    terraform_state_bucket = arn:aws:s3:::rao-ksm-chartmuseum
    {% endhighlight %}

You can now use the information on he output section above to interact with the bucket. Remember , you will need to use the credentials used to create the bucket or some other credentials that have read or read/write access to the bucket. 
The Source code used above can be found on my [git repo](https://github.com/honnuanand/s3-terraform.git).

