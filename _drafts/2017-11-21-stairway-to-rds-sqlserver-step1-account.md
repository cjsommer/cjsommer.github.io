---
layout: post
title: Stairway to Amazon's RDS for SQL Server - Account Setup
date: 2017-11-21 00:00
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
<!-- Image and URL references used in this post -->
[url_terraform]: https://www.terraform.io/
[url_aws_root]: https://aws.amazon.com/
[url_aws_billing_alarm]: http://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier-alarms.html
[img_aws_billing_alarm]: /img/2017/10/aws_billing_alarm.png
[url_aws_free_signup]: https://aws.amazon.com/free/
[img_aws_free_signup]: /img/2017/10/aws_free_signup.png
[url_aws_create_iam_user]: https://console.aws.amazon.com/iam/home#/users
[img_aws_create_iam_user]: /img/2017/10/aws_create_iam_user.png

<!-- Content -->
# Stairway to RDS for SQL Server
So you want to run SQL Server in Amazon Web Services Relational Database Service (AWS/RDS)? Amazon has done a great job providing SQL Server as a service through their RDS platform. One of the main attractions of moving to a PaaS type offering is that a lot of infrastructure management is handled by the cloud provider. In RDS Amazon takes care of the underlying hardware, the operating system (including patching), minor SQL Server version upgrades and last but certainly not least your SQL Server backups. Those are some really attractive reasons to move to the cloud, but they don't come without a cost and a few limitations. Managing SQL Server in RDS isn't quite as hands-off as you might think. 

This blog series is aimed at someone new to AWS who is interested in running SQL Server in RDS. Here's a general idea of all of the topics I plan to cover, although it may be a little fluid based on user feedback and such.

- Signing up and configuring an AWS IAM user (this post)
- Creating an RDS SQL Server using the AWS console
- Configuring an RDS SQL Server
- Monitoring an RDS SQL Server
- Using the AWS cli
- Using AWSPowerShell
- DevOps - Using [Terraform][url_terraform] to manage your AWS/RDS Infrastructure
- TBD...

## AWS common terms and acronyms
To start lets get some common acronyms and terminology out of the way. It'll save me a lot of typing later on, and it'll get you on your way toward speaking AWS.

- AWS - Amazon Web Services. Amazon's entire cloud platform.
- CloudWatch - Alarms, Events and Metrics related to your AWS resources.
- IAM - Identity and Access Management 
- RDS - Amazon's Relational Database Services. This blog series will be focused on SQL Server, but RDS also offers a number of other platforms for your database needs. (SQL Server, Oracle, MySQL, MariaDB, Aurora, Postgres)
- VPC - Virtual private cloud. An elastic network populated by infrastructure, platform, and application services that share common security and interconnection.
- EC2 - A web service that enables you to launch and manage Linux/UNIX and Windows server instances in Amazon's data centers.

[url_aws_terminology]: http://docs.aws.amazon.com/general/latest/gr/glos-chap.html

Some of those are directly from [Amazon's Terminology Glossary][url_aws_terminology] which contains definitions for all of the acronyms and terminology you will encounter while working with AWS.

# Step 1 - Sign up for an AWS Free Tier Account
Now that we have some of the AWS terminology out of the way, lets start by signing up for an AWS Free Tier account. It's a perfect playground for learning AWS and gives you 12 months of access to their free tier services. If you already have an Amazon account then you should be able to use that. If not, you'll have to create a new one.

Just go to the [AWS Free Tier Signup Page][url_aws_free_signup] to follow the sign up process. If you want more details as to what you get with the free tier account you'll be able to find those details on this page as well.

[![AWS Signup Page][img_aws_free_signup]][url_aws_free_signup]

# Step 2 - Setup your billing alarm
**Disclaimer!** If you spin up resources in AWS that are not part of the free tier, or over the limits as defined in the agreement, you will be charged for them. From my experience Amazon is pretty good about warning you before you do that, but just wanted to make you aware of this. You can [set up billing alarms][url_aws_billing_alarm] for your account and I highly recommend doing so. This is a nice safety net and should help prevent unwanted charges to your credit card. 

[![Billing Alarm][img_aws_billing_alarm]][url_aws_billing_alarm]

# Step 3 - Create an IAM user (optional)
When you signed up for the AWS account you did create an initial user. That is what is known as the root user. It has full access to all things inside AWS, including billing. This user should be protected and only used when absolutely necessary. One practice I follow is to never use the root user for day-to-day care and feeding of my AWS environment. So how do you do this, you may ask? It just so happens that AWS gives you the ability to create additional users for managing your AWS resources. These are called IAM users. You also have access to IAM roles to help manage Role Based Access Control (RBAC) in your AWS subscription. This gives us very granular control over privileges. 

I marked this step as optional because if you're the only person using your AWS account then it really doesn't matter, but I think it's nice to get used to some of the recommended practices right out of the gate.

You can create IAM users from the [IAM User Creation][url_aws_create_iam_user] screen. Just click the "Add User" button and follow the wizard to create an IAM user. 

[![IAM User Creation][img_aws_create_iam_user]][url_aws_create_iam_user]

# Conclusion
So that's it for episode 1 of the Stairway to AWS series. At this point you have an AWS account (free tier), billing alarms to make sure you don't get any surprise bills, and an IAM user to manage your account. You're on your way. Stay tuned because next time we'll be firing up our first RDS SQL Server instance! Good times!

## Series Index
- Signing up and configuring an AWS IAM user (this post)
- Creating an RDS SQL Server using the AWS console
- Configuring an RDS SQL Server
- Monitoring an RDS SQL Server
- Using the AWS cli
- Using AWSPowerShell
- DevOps - Intro to using [Terraform][url_terraform] to manage your RDS Infrastructure
- TBD...

