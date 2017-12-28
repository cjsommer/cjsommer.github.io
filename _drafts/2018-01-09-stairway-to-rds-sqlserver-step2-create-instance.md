---
layout: post
title: Stairway to RDS for SQL Server - Create an RDS Instance
date: 2018-01-09 00:00
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

In Step 1 of the series we learned to bla foo bla.

Now lets learn how to fire up our own SQL Server in RDS


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