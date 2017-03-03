---
layout: post
title: T-SQL Tuesday #73 – Naughty or Nice?
date: 2015-12-08 21:43
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
<img src="http://www.cjsommer.com/wp-content/uploads/2015/05/TSQLTuesday.jpg" alt="TSQLTuesday" width="150" height="150" class="alignright size-full wp-image-504" />
Thank you to SQLBalls ( <a href="http://www.sqlballs.com/"  target="_blank">b</a> | <a href="https://twitter.com/SQLBalls" target="_blank">t</a> ) for hosting this month's #tsql2sday. 

<em>As you work with SQL Server look around you.  Is your environment Naughty or Nice?  If it is Naughty what’s wrong with it?  What would you do to fix it?  Do you have a scrooge that is giving you the Christmas chills?  Perhaps you have servers of past, present, and future haunting you.  Maybe you are looking at SQL Server 2016 like some bright shining star in the east.</em>
<hr>
<h2>The Problem: Disk Space Alert</h2>
You've just received a call for a disk space alert. It appears as though the drive that holds your backup files has grown to over 90% full.

Naughty DBA's would manually delete a few of the bigger backup files to get the free space down to an acceptable level and go back to bed, leaving a potential time bomb for the next DBA.

Nice DBA's always dig a little deeper. Maybe the backup jobs weren't setup to remove old backup files like they were supposed to. The nice DBA would ensure that the backup jobs were set to automatically remove old backups like they are supposed to. Or maybe a new database was added and nobody expanded the backup drive to accommodate. The Nice DBA would request additional storage.
<hr>
<h2>The Problem: Failed SQL Agent Job</h2>
You received a call from the operations center for a SQL Agent job failure. 

Naughty DBA's first reaction is to rerun the job, and with a little bit of luck, the job runs fine when run manually. Naughty DBA goes back to bed...

Nice DBA's dig a little deeper. They look into the SQL Agent job history to find that this same job has been failing every night for the past couple weeks, and the resolution has been to just rerun the job. Looking at the job schedule they notice that it is running at exactly the same time as another SQL Agent job, so they decide to move the schedule by a minute, and sure enough the job failures go away.

<hr>
These are very specific examples, both of which I have personally experienced. I hate finding time bombs. Alerts or issues that other people have worked on and opted to just treat the symptoms. Sure enough they come back to bite you in the ass, every...single...time. Treating the symptoms and alleviating the pain is always a good first step, but that's never where your investigation and problem resolution should end. Determining root cause is a huge step towards building a stable environment. 

So be a nice DBA! Your teammates will thank you and you'll probably get something really sweet from the big guy!
<img alt='' class='alignnone size-full wp-image-1166 ' src='http://www.cjsommer.com/wp-content/uploads/2015/12/img_5667870c3d4c1.png' />
