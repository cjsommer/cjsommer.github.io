---
layout: post
title: T-SQL Tuesday #66 - Monitoring
date: 2015-05-12 08:25
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
Thank you <a href="http://www.cathrinewilhelmsen.net/2015/05/05/invitation-to-t-sql-tuesday-66-monitoring/">Cathrine Wilhelmsen</a> for hosting this week's #tsql2sday.

If I knew that this T-SQL Tuesday was going to be about monitoring I would have saved my <a href="http://www.cjsommer.com/mrkrabs-sqlagent-job-monitoring/" target="_blank">Monitoring SQL Agent Jobs</a> post from last month. No worries though! I think I'll talk more generally about monitoring. 

<h2>If you don't monitor anything else...</h2>
<ul>
	<li><strong>Up/Down Monitoring</strong></li> Are my servers and/or SQL services up or down? If SQL Server is down, none of the other stuff really matters.
	<li><strong>Capacity Monitoring</strong></li> How are my physical resources? Am I running out of disk space? Is my CPU running hot all the time? How is my system memory doing and am I paging at all?
	<li><strong>SQL Agent Job Monitoring</strong></li> Are all of my SQL Server Agent jobs such as backups, index rebuilds and integrity checks running successfully every day?
</ul>
<hr>
If you don't monitor anything else in your SQL Server environment, you need to monitor these things. They encompass the foundation of a DBA's responsibilities, availability and recoverability. Yes there are other things that we can monitor like SQL Server internals and query performance, but those are kinda pointless if your SQL Server is unavailable for any reason.

If you read my <a href="http://www.cjsommer.com/mrkrabs-sqlagent-job-monitoring/" target="_blank">Monitoring SQL Agent Jobs</a> post from last month you will know that I had just begun a new position that had no monitoring. We also didn't have a budget to go out and buy a solution, which is precisely what motivated me to learn PowerShell. I needed to know when my SQL Servers were having a problem so I decided to script my own solutions. If you are working with SQL Server you already have all of the tools you need to put together your own custom monitoring solutions. 

Take an inventory of your monitoring. Are you covering the basics for availability and recoverability? Are you confident that you will know when you are running out of disk space? Do you know when your SQL Server is down before your users do? Do you know when your backup jobs are failing? If you answered no to any of these questions, I highly recommend developing a plan to fill those gaps. There is peace of mind in having a solid monitoring solution in place. Monitoring isn't really an option when it comes to being a DBA, it is a must have.
