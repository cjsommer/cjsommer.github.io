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
<!-- Content -->
# Stairway to RDS for SQL Server

## Series Index
- Signing up and configuring an AWS IAM user
- Creating an RDS SQL Server using the AWS console
- Managing RDS SQL Servers using PowerShell
- CloudWatch metrics through the AWS console (?)
- Gathering CloudWatch performance metrics with PowerShell
- DevOps - Intro to using [Terraform][url_terraform] to manage your RDS Infrastructure

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


# RDS Vernacular Used in this Series
- AWS
- RDS
- IAM
- CloudWatch