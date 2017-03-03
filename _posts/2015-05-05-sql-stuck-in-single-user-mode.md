---
layout: post
title: Help! My database is stuck in SINGLE_USER mode!
date: 2015-05-05 08:15
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
[caption id="attachment_284" align="alignright" width="272"]<img src="http://www.cjsommer.com/wp-content/uploads/2015/04/StuckInSingleUser.png" alt="HELP! My database is stuck in single user mode!" width="272" height="146" class="size-full wp-image-284" />HELP! My database is stuck in single user mode![/caption]

I answered this twitter post a while back and figured it would make a fun blog post.

So what do I mean that the database is stuck in single user mode? Simply speaking, it means that the database is in single user mode and you can't seem to get it back into multi user mode. As an example I set my local AdventureWorks2012 database to single user mode, opened a session to that database, tried an alter database to get it back to multi user and this is what I got.

[caption id="attachment_400" align="alignnone" width="796"]<img src="http://www.cjsommer.com/wp-content/uploads/2015/05/SingleUser_1.jpg" alt="Yup! Most definitely stuck. Won&#039;t go back to MULTI_USER!" width="796" height="298" class="size-full wp-image-400" /> Yup! Most definitely stuck. You can see from the error message that it won't go back to MULTI_USER![/caption]

I know I have gotten myself into this situation more than once. For whatever reason I thought it would be a good idea to put the database in single user mode, and depending on database activity, it caused me a minor headache to get it back out. If it's the first time you've ever done it, it can definitely be one of those "oh crap" moments. You know the one. Where all the blood drains out of your face, you start to sweat and you wonder if you'll still have a job in the morning. Yeah, that one. 

I'm here to tell you that your predicament is not that dire. It usually it means that there is another session to that database preventing your session from performing your alter database, and the most typical resolution is to find the session that has control of your database and kill it! 

Here is the connection that has control of my AdventureWorks2012 database.

[caption id="attachment_409" align="alignnone" width="754"]<img src="http://www.cjsommer.com/wp-content/uploads/2015/05/SingleUser_2.jpg" alt="This is the session holding the one connection to my AdventureWorks2012 database." width="754" height="219" class="size-full wp-image-409" />This is the session holding the one connection to my AdventureWorks2012 database.[/caption]

Here is the TSQL used to find the session that you need to kill.

<pre class="theme:ssms2012 lang:tsql decode:true " title="Find spid's connected to your database" >
USE [master] ;
 
DECLARE @DatabaseName VARCHAR(255) ;
SET @DatabaseName = 'AdventureWorks2012' ;
 
SELECT spr.spid AS [spid]
     , sdb.NAME AS [DatabaseName]
     , spr.open_tran AS [OpenTransactions]
     , spr.status AS [Status]
     , ('kill ' + CAST(spr.spid AS VARCHAR)) AS [KillCommand]
FROM sys.sysprocesses spr
INNER JOIN sys.databases sdb ON sdb.database_id = spr.dbid
WHERE sdb.NAME = @DatabaseName ;
</pre> 

And here are the results from the TSQL.

[caption id="attachment_411" align="alignnone" width="504"]<img src="http://www.cjsommer.com/wp-content/uploads/2015/05/SingleUser_3.jpg" alt="Here are the results from the session find script, and you can see that is calls out SPID 53." width="504" height="344" class="size-full wp-image-411" />Here are the results from the session find script, and you can see that is calls out SPID 53.[/caption]

So at this point I know I have to kill SPID 53 so I can proceed with getting my database back into multi user mode. I don't like having my script automatically kill it in case there are open transactions. Yes it adds another step to my process, but if there is a legitimate transaction open I may want to consider what it is doing before I go killing it. 

As you can see SPID 53 has no open transactions so I determine that I am OK to kill it at this point.

[caption id="attachment_413" align="alignnone" width="703"]<img src="http://www.cjsommer.com/wp-content/uploads/2015/05/SingleUser_4.jpg" alt="And as you can see after killing SPID 53 I was able to set it back into multi user mode." width="703" height="295" class="size-full wp-image-413" />And as you can see after killing SPID 53 I was able to set it back into multi user mode.[/caption]

And here we are back in business! I have never had this not work. I have had some issues with a very persistent application trying to connect before I was able to complete the process of finding the spid, killing it and altering my database, but it has always worked for me. @sqL_handLe did mention that Async Statistics could also possibly consume a connection, and that might not show up in the session results using the query above, but I have not seen this problem so I cannot speak from experience on that. Just another good to know tidbit.

So if you end up getting your database stuck in single user mode, don't worry, there's usually a way out.

