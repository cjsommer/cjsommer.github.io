---
layout: post
title: T-SQL Tuesday 89 - The times they are a changing!
date: 2017-04-11T10:00:00
author: cjsommer@gmail.com
comments: true
tags: ["SQL Server"]
---
T-SQL Tuesday is a monthly blog party for the SQL Server community (or Microsoft Data Platform community. Although it’s called T-SQL Tuesday, it’s not limited to SQL Server database engine only). It is the brainchild of Adam Machanic [blog](http://sqlblog.com/blogs/adam_machanic/) &#124; [twitter](https://twitter.com/AdamMachanic).

<img src="/img/TSQLTuesday.jpg" alt="tsql2sday" align="right">

This month's tsql2sday is being hosted by Koen Verbeeck [blog](http://sqlkover.com/t-sql-tuesday-89-invitation-the-times-they-are-a-changing/) &#124;[twitter](https://twitter.com/Ko_Ver) and the topic is "The times they are a changing". Basically, how are you handling the ever changing landscape of the SQL DBA. It's a brave new world and the days of the point-n-click, drag-n-drop administrator are fleeting. Installing SQL Server. Checking backup. Performing health checks. A Jedi-DBA craves not these things!

---

So what are some of the new roles we find ourselves in?

DevOps DBAs! DBAs in today's workforce are building tools and automating processes to help provide services to their customers. DevOps is an interesting term which in my mind tries to bridge that gap between developer and operations. We're no longer performing the mundane tasks of old. DBAs are becoming database engineers and developing solutions to problems, not just administering databases. As someone who embraced PowerShell and the automation mindset early on in my career, I personally love this transformation. Learning PowerShell (or some other scripting language) is paramount to success for the DevOps DBA.

Cloud DBA! Azure SQL Database, AWS, Google Cloud, etc. It's cheaper, right? As a good DBA my answer is "it depends". Running SQL Server in the cloud isn't quite the same as running SQL Server in house. In Azure it feels like you're running in a partially contained database (and for good reason). A lot of what you know from being a SQL DBA carries into the cloud, but there's a whole new set of tools that you have to learn to manage your databases in the cloud. I can't speak for AWS quite yet, but it looks like I'll have an opportunity to learn that very shortly. Definitely looking forward to learning a new platform for SQL Server. Embrace the cloud. It isn't going away any time soon!

Linux SQL Server DBA! Penguins, penguins, everywhere! We're in the middle of a penguin invasion and it's all Microsoft's fault. SQL Server vNext [running on Linux](https://www.microsoft.com/en-us/sql-server/sql-server-vnext-including-Linux). PowerShell open sourced and [running on Linux](https://azure.microsoft.com/en-us/blog/powershell-is-open-sourced-and-is-available-on-linux/). .NET also [running on Linux](https://www.microsoft.com/net/core#linuxredhat). Who would have thought Microsoft would have glommed onto the the Linux craze? Definitely not me! I started my career as a DBA working with Informix and Oracle on the unix platform and I actually enjoyed working in that environment. Fundamentally both Windows and Linux provide the same thing; a platform for delivering database services to your customers. Although I haven't had the time to dive into SQL on Linux yet, it's on my short term roadmap of things to learn and I'm definitely looking forward to it.

It's a constantly changing world for a DBA. New platforms, new technologies, finding more efficient ways to do things, unlearning old habits. Embrace the change and learn, learn, learn. There is so much opportunity for growth right now. If you're not learning in this career field, you're doing it wrong.