---
layout: post
title: What I Learned Today - Configuring tempdb in RDS
date: 2017-08-28 00:00
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---

# Today I learned that you can configure tempdb in RDS! 

One of the things we DBA's do is configure tempdb to have multiple data files (if needed). Not going to go into the depths of why in this post, but adding multiple data files can help alleviate contention in tempdb. Until today I didn't think you could do this in RDS. I guess I hadn't even tried because you can't do anything to master, model or the msdb databases, so I just assumed tempdb was off limits as well. 

Never assume anything because it makes an ![DONKEY!](/img/2017/08/donkey.gif) out of you and me! 

I was working on a presentation for tomorrow's AWS meetup and part of my presentation is about configuring SQL Server, so I decided to check my assumptions about tempdb. I decided to try to add a second data file to tempdb and sure enough  I was able to.

## Display the default (initial) state of tempdb on an RDS instance
```sql
-- Display database_files
use [tempdb] ;

SELECT file_id
	, type_desc
	, name
	, physical_name
	, state_desc
FROM sys.database_files ;
```

![Database Files Before](/img/2017/08/database_files_before.png)

## Add a new tempdb data file (tempdev1)
```sql
-- Add a new tempdb data file
USE [master] ;

ALTER DATABASE [tempdb] ADD FILE (
	NAME = N'tempdev1'
	,FILENAME = N'D:\RDSDBDATA\DATA\tempdev1.ndf'
	,SIZE = 8192 KB
	,FILEGROWTH = 65536 KB
	) ;
```

## Display tempdb data and log files after the data file addition
![Database Files After](/img/2017/08/database_files_after.png)

# Lesson Learned!
One of my favorite things about creating a new presentation is challenging my own beliefs and assumptions. I learn something every.single.time!