---
layout: post
title: Stairway to RDS Series for SQL Server
date: 2017-09-15 00:00
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
So you want to run SQL Server in Amazon Web Services? There are a number of reasons why people look at the cloud, but that's not really the focus of this blog series. I am hoping to cover the basics to help get you up and running in RDS. I know one of the main attractions of moving to the cloud seems to be that it is managed by Amazon. You don't have to worry about anything and Amazon know best, right? Well sort of. Amazon does a lot of really great things for you, but that doesn't come without a cost. Managing SQL Server in RDS isn't quite as hands-off as you might think, and if you've been working with SQL Server for any length of time you might find it a bit different than running your own servers.

To start lets get some common acronyms and terminology out of the way. It'll save me a lot of typing later on, and it'll get you on your way toward speaking AWS.

## AWS common terms and acronyms
- AWS - Amazon Web Services. This is the terms that encompasses all of their offerings.
- CloudWatch - Alarms, Events and Metrics related to your AWS resources
- IAM - Identity and Access Management 
- RDS - Amazon's Relational Database Services. This blog series will be focused on SQL Server, but RDS also offers a number of other platforms for your database needs. (SQL Server, Oracle, MySQL, MariaDB, Aurora, Postgres)

# Step 1 - Sign up for an AWS Free Tier Account
Now that we have some of the AWM terminology out of the way, lets start by signing up for an AWS Free Tier account. It's a perfect playground for learning AWS and gives you 12 months of access to their free tier services. If you already have an Amazon account that you use to purchase things then you should be able to use that. If not, you'll have to create a new one.

Just go to the [AWS Free Tier Signup Page][url_aws_free_signup] and sign up. If you want more details as to what you get with the free tier account you'll be able to find those details on this page as well.

[![AWS Signup Page][img_aws_free_signup]][url_aws_free_signup]

# Step 2 - Setup your billing alarm
**Disclaimer!** If you spin up resources in AWS that are not part of the free tier, or over the limits as defined in the agreement, you will be charged for them. From my experience Amazon is pretty good about warning you before you do that, but just wanted to make you aware of this. You can [set up billing alarms][url_aws_billing_alarm] for your account and I highly recomend doing so. This is a nice safety net and should help prevent unwanted charges to your credit card. 

[![Billing Alarm][img_aws_billing_alarm]][url_aws_billing_alarm]

# Step 3 - Create an IAM user (optional)
When you signed up for the AWS account you did associate that with an Amazon user. That is what is known as the root user. It has full aceess to all things inside AWS, including billing. This user should be protected and only used when absolutely necessary. One of the best practices I follow is to never use the root user for day-to-day care and feeding of my AWS environment. We do that using IAM users. IAM users are just additional users I can create and grant permissions to inside my AWS environment. This gives us very granular control over priviliges. 

I marked this step as optional because if you're the only person using your AWS account then it really doesn't matter, but I think it's nice to get used to some of the recommended practices right out of the gate.

You can create IAM users from the [IAM User Creation][url_aws_create_iam_user] screen.
[![IAM User Creation][img_aws_create_iam_user]][url_aws_create_iam_user]

# Amazon's Relational Database Service (RDS)
Amazon Relational Database Service is Amazon Web Service's database platform as a service offering. There are 5 different relational database platforms offered.

|Edition|Description|
|-------|-----------|
|Amazon Aurora|Amazon Aurora is a MySQL- and PostgreSQL-compatible|
|MySQL Community Edition|MySQL is the most popular open source database in the world. MySQL on RDS offers the rich features of the MySQL community edition with the flexibility to easily scale compute resources or storage capacity for your database.|
|MariaDB Community Edition|MariaDB Community Edition is a MySQL-compatible database with strong support from the open source community, and extra features and performance optimizations.|
|PostgreSQL|PostgreSQL is a powerful, open-source object-relational database system with a strong reputation of reliability, stability, and correctness.|
|Oracle|Standard and Enterprise Editions|
|Microsoft SQL Server|Express, Web, Standard, and Enterprise Editions|


## Series Index
- Signing up and configuring an AWS IAM user
- Creating an RDS SQL Server using the AWS console
- Managing RDS SQL Servers using PowerShell
- CloudWatch metrics through the AWS console (?)
- Gathering CloudWatch performance metrics with PowerShell
- DevOps - Intro to using [Terraform][url_terraform] to manage your RDS Infrastructure