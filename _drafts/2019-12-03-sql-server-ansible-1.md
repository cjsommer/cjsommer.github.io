---
layout: post
title: Ansible for the SQL Server DBA | Part 1 | Why Ansible
date: 202019-12-03 17:00
author: cjsommer@gmail.com
comments: true
categories: [automation]
---
<!-- Image and URL references used in this post -->
[url_ansible]: https://www.ansible.com/
[url_ansible_wikipedia]: https://en.wikipedia.org/wiki/Ansible_(software)
[url_saltstack]: https://www.saltstack.com/
[url_puppet]: https://puppet.com/
[url_chef]: https://www.chef.io/
[url_choco]: https://chocolatey.org/
[url_terraform]: https://www.terraform.io/
[url_cloudformation]: https://aws.amazon.com/cloudformation/
[img_ansible_logo]: /img/2019/12/ansible-logo.png
[url_sql_ansible_1]: /2019-12-04-sql-server-ansible-1/

# Ansible for the SQL Server DBA | Part 1 | Why Ansible
---
[![Ansible][img_ansible_logo]][url_ansible]

***Ansible for the SQL Server DBA*** will be a series about my adventures with this amazing tool. The end goal is walk you through all of my discoveries so that you will be able to install and configure SQL Server with Ansible. Looking forward to documenting my journey and am sure I'll be learning more along the way!

## My Disclaimer
---
> I am still learning and very much a novice when it comes to using Ansible. I have the basics down, and I've already learned enough to make it useful for me, but I know I still have a lot to learn. I would by no means consider myself an Ansible guru. Writing about topics that I am not 100% confident in has always been a great way for me to learn and fill in the missing pieces. If you decide to continue reading (and I hope you do), just know that you'll be learning right along side me!

## What exactly is Ansible?
---
[Ansible][url_ansible] is an automation tool that provides centralized deployment and configuration of software across a heterogenous environment. It fills the configuration management piece of the "infrastructure as code" puzzle. 

Here is the definition from the Ansible From [Wikipedia][url_ansible_wikipedia]: 

> Ansible is an open-source software provisioning, configuration management, and application-deployment tool. It runs on many Unix-like systems, and can configure both Unix-like systems as well as Microsoft Windows. It includes its own declarative language to describe system configuration.
>
> Ansible was written by Michael DeHaan and acquired by Red Hat in 2015. Ansible is agentless, temporarily connecting remotely via SSH or remote PowerShell to do its tasks.

Other tools that I would consider similar to Ansible would be [Puppet][url_puppet], [Chef][url_chef], [SaltStack][url_saltstack]. There are a number of other ones out there too, but I think these are the front runners. 

So why Ansible instead of the others? Well for me it was mostly because the DevOps team at my company was already using it. I guess one thing I like about Ansible compared to the other tools is that it is agentless. Ansible uses WinRM (PowerShell Remoting) for managing a Windows server. Puppet, Chef, and SaltStack all require an agent on the server you want to manage, but that definitely wouldn't be a deal breaker by any means. Like I said, my choice to go with Ansible was to align with my DevOps team which had already headed down that road.

## What is Ansible good for?
---
Configuring a server 
- Configure networking
- Joining a domain
- Setting hostname 

Installing software
- [Chocolatey][url_choco]
- MSI's
- EXE's

Configuring software
- SQL Server

## What is Ansible not good for?
---
After a good bit of reading I have come to believe that Ansible isn't all that good as an infrastructure provisioning tool. I guess it can do cloud provisioning, but tools like [Terraform][url_terraform], [AWS CloudFormation][url_cloudformation], or sysprep images, are built for and better suited to handle infrastructure provisioning. It seems like a lot of people pair a provisioning tool (Terraform) with a Configuration Management tool (Ansible) for an end to end solution. 

This blog series will be showcasing Ansible as a configuration management tool. Provisioning will be mentioned briefly a few times, but it will not be the focus. 

## Why should a DBA care about a configuration tool like Ansible? What can I do with it?
---
Ansible is an excellent way to install and configure SQL Server. 

I mean seriously, how hard is it to install and configure SQL Server? 

Double click -> Next -> Next -> Next -> Reboot -> Done

Right?!?!

Well if you've been a DBA for any length of time you know it's really not that simple. You have to set service accounts, grant local security policy permissions, default file locations, etc, etc, etc. And that's just during installation. And after you install SQL Server you still have to configure it. You have to set Min/Max memory, MaxDOP, Cost Threshold for Parallelism, Backup Compression Default, etc, etc, etc. Yes you can do all of those things through the GUI. Yes you can do all of those things through scripts (TSQL, PowerShell). And yes you can also do them with tools like Ansible.

Unattended installations are not new. We have command line installation with an INI file (wrapped in cmd or PowerShell), and DSC (not well adopted as far as I can tell by reading). But I also know that a lot of people fall back to the GUI, especially if installing SQL Server isn't something you do very often. Installing SQL Server with Ansible gives you that same level of consistency any of the other automated methods do, and it does it right along side all of your other infrastructure. There's value in centralized management.

I think configuration is a totally different ballgame. I'm sure there are some outliers, but just from my experience I think most people set it and forget it when it comes to configuring their SQL Server. Maybe they use a script for the initial configuration, but what about changes over time? How do you keep track of that? If you ever have to rebuild it, how do you know what to build? That is one of the most amazing things about using a tool like Ansible for managing your SQL Server configurations. You have your SQL Server defined "as code" and if you are rigid in your processes you will always know how to rebuild it.

## Ansible for the SQL Server DBA
---
- Part 1 | Why Ansible
- Part 2 | Getting Started (up next)
