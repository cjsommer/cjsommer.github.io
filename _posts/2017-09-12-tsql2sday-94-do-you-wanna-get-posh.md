---
layout: post
title: T-SQL Tuesday 94 - Do you wanna get PoSh?
date: 2017-09-12 00:00
author: cjsommer@gmail.com
comments: true
draft: true
categories: [SQL Server]
---
<!-- Image and URL references used in this post -->
[img_tsql2sday_logo]: /img/TSQLTuesday.jpg
[url_am_blog]: http://sqlblog.com/blogs/adam_machanic/
[url_am_twitter]: https://twitter.com/AdamMachanic
[url_rob_blog]: https://sqldbawithabeard.com/2017/09/05/tsql2sday-94-lets-get-all-posh/?utm_content=buffere68cc&utm_medium=social&utm_source=twitter.com&utm_campaign=buffer
[url_rob_twitter]: https://twitter.com/sqldbawithbeard
[url_dbatools]: https://dbatools.io/
[url_sqlslack]: https://sqlcommunity.slack.com/

<!-- Content -->
<div style="text-align:right"><img src ="/img/TSQLTuesday.jpg" /></div>

T-SQL Tuesday is a monthly blog party for the SQL Server community (or Microsoft Data Platform community. Although it’s called T-SQL Tuesday, it’s not limited to SQL Server database engine only). It is the brainchild of Adam Machanic ([blog][url_am_blog] | [twitter][url_am_twitter]).

This month's T-SQL Tuesday is being hosted by Rob Sewell ([blog][url_rob_blog] | [twitter][url_rob_twitter]). From his blog post:

_This month’s topic: Let’s get all Posh – What are you going to automate today?_

_It is no surprise to those that know me that I will choose PowerShell as the topic for this month. I am passionate about PowerShell because it has enabled me to have the career I have today and to visit numerous countries all around the world, meet people and talk about PowerShell. By my reckoning searching the TSQL Tuesday website it has been over 3 years since we had a topic specific to PowerShell. So I would like you to blog about PowerShell and SQL Server (or other interesting data platform products)_

# Love me some PowerShell!
Just like Rob, I am also passionate about PowerShell. I love it! I have been using it since PowerShell 2.0 came out and it's been my goto tool for automation and scripting ever since. But before I get to some of the things I've done with PowerShell let me tell you how I got here.

## My dark past on the Unix platform
I've been a DBA for about 20 years. The first 10 of those years were spent working with Informix and Oracle databases on the Unix platform (specifically AIX, Solaris and Linux). The cool thing about being a DBA is that we perform a lot of the same tasks regardless of the platform we are working on. For example:
- System builds
- Backup and recovery
- Updating statistics and index maintenance
- Refreshing development systems
- Database migrations

The list goes on and on. The major difference I found working on Unix was the tooling. A lot of the graphical tools that we are used to having with SQL Server didn't exist or weren't as mature on Unix. Because of this I became best friends with the Unix shell, mainly ksh and bash. I was very fortunate to be a part of a very strong team who helped me learn a ton about scripting. Anything we had to do with Informix and Oracle was done using shell scripts, and we automated everything. So needless to say I got pretty good at scripting.

## Fast forward 10 years...SQL Server here I come!
About 10 years into my career as a DBA I had a great opportunity present itself. The biggest change for me was that it was more focused on SQL Server, a new platform for me. I decided to take that opportunity and SQL Server has been my primary platform ever since.

Of course one of the first things I looked for when I entered Windows Land was something similar to what I had been doing on Unix. Luckily for me PowerShell was starting to gain steam and it was a pretty natural fit for me. There are plenty of differences between PowerShell and bash, but the feel of the language made me feel right at home. I was up and running with PowerShell in no time.

## Do You Wanna Get Posh?!?!
<div style="text-align:right"><img src ="/img/do_you_wanna_get_rocked.jpg" /></div>
So what can you do with PowerShell? I think the best way for me to answer is by telling you what I have done with PowerShell.

Much of my early PowerShell scripting was simply gathering information about my SQL Servers. My bread and butter has to be the SQL Management Object libraries (a.k.a. SMO). SMO is a set of .NET libraries for interacting with SQL Server. Anything you can do using SQL Servers graphical tools (SSMS, SQL Configuration Manger) you can do using SMO and PowerShell. This has been a foundation of pretty much everything I've done with PowerShell as a SQL DBA. As time went on the tools I have built became more complex.

A few things that I have done:
- SQL Agent Job monitoring and daily report
- Weekly schema dumps using the SMO scripter object
- Weekly schema comparisons using RedGate's SQL Compare command line
- Dumping SQL logins and group permissions as well as group membership from active directory
- Automatic SQL Server start & stop scripts
- Multiple database deployment tools for releasing database code (SQL scripts, SQL Server Data Tools)
- A tool to migrate our databases (20,000+) from SQL 2005 to SQL 2012. This tool was built around database mirroring and was fully automated.
- A tool to migrate 15,000 databases from on-prem to Microsoft Azure. Once again a fully automated, database driven solution.
- A migration tool for Amazon RDS (currently in progress)

The list goes on and on and on.

I know it can be hard to get started and sometimes the hardest part is picking what to you want to do. If that's the case then maybe one of my projects can spark an idea for you. I also contribute to the [dbatools.io][url_dbatools] project when I have time, and were always looking for new contributors so swing on by if you're looking for ideas. We'd love to have you join us in the powershell channel on the [SQL Server Community Slack][url_sqlslack] page. Lots of really awesome people and experienced PowerShell programmers frequent there in case you're looking for help.

Scripting and automation can be very rewarding, and the benefits you reap in the reliability and stability of your environment can be significant. If you've been standing on the water's edge, the time to dive on in is now! I promise there are no sharks in this water, only a great community willing to help you succeed.
