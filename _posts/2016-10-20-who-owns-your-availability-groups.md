---
layout: post
title: Who owns your availability groups?
date: 2016-10-20 09:00
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
<h3>Question: Who owns your availability groups?</h3>
The person who creates the AG becomes the owner by default. Did you know that you can (and probably should) change that the same as many of the other SQL Server objects like databases, jobs, endpoints, etc. Up until this week I did not know that you could change AG owner principal, and based on my internet searches I'm not sure a lot of other people do either. That's the general gist of this blog post.

Availability groups have owners, just like every other object in SQL Server. The owner sid is found in sys.availability_replicas. The script below will map those sids to an actual principal name, and display the owners for all AG's on the replica where you run it. You can run this on both the PRIMARY and SECONDARY replicas, which we have discovered may or may not be consistent.  Regardless, you should see an owner for every AG on your SQL Server. 

```sql
SELECT ar.replica_server_name
	,ag.name AS ag_name
	,ar.owner_sid
	,sp.name
FROM sys.availability_replicas ar
LEFT JOIN sys.server_principals sp
	ON sp.sid = ar.owner_sid 
INNER JOIN sys.availability_groups ag
	ON ag.group_id = ar.group_id
WHERE ar.replica_server_name = SERVERPROPERTY('ServerName') ;
```

<img alt='' class='alignnone size-full wp-image-1416 ' src='/img/2016/10/img_5808d8ed4c4aa.png' />

<hr>
<h3>Question: Why should I care?</h3>
Up until last week my answer probably would have been "who cares". But last week we were trying to configure least privilege for an application account that required VIEW DEFINITION on an AG...and this was the result. 

<img alt='' class='alignnone size-full wp-image-1366 ' src='/img/2016/10/img_58075c7754ab5.png' />

Those severe errors are a <strong>stack dump</strong> (which I won't post here). There wasn't a lot of useful information in the dump text that we could interpret so we opened a ticket with Microsoft. They told us our error was because our AG owner principal was set to NULL (i.e. it didn't have an owner defined). Huh, weird. Lets check.

```sql
SELECT ar.replica_server_name
	,ag.name AS ag_name
	,ar.owner_sid
	,sp.name
FROM sys.availability_replicas ar
LEFT JOIN sys.server_principals sp
	ON sp.sid = ar.owner_sid 
INNER JOIN sys.availability_groups ag
	ON ag.group_id = ar.group_id
WHERE ar.replica_server_name = SERVERPROPERTY('ServerName') ;
```

<img alt='' class='alignnone size-full wp-image-1417 ' src='/img/2016/10/img_5808d958c76da.png' />

Sure enough, just like they said. Our owner was NULL for some reason. At this point Microsoft recommended that we reset the AG owner to 'sa' and we'd be on our way. So off we went to MSDN to find the commands...nothing to be found about changing availability group ownership there. So then off we went to Google where we found the end of the internet. 

<img alt='' class='alignnone size-full wp-image-1408 ' src='/img/2016/10/img_5807f7578d0ff.png' />

Hmm, that's strange, no help on the internet for changing the AG principal.  So we once again decided to ask Microsoft and they provided us with an <strong>undocumented command</strong> for setting the AG owner principal. Apparently they just haven't added it to their documentation yet. (Thanks @sqlsoldier <a href="http://www.sqlsoldier.com/" target="_blank">b</a>|<a href="https://twitter.com/SQLSoldier" target="_blank">t</a> for reaching out to Microsoft for confirmation)

```sql
ALTER AUTHORIZATION ON AVAILABILITY GROUP::TESTAG to [sa] ;
```

Simple fix. Just plug in our AG info, run the T-SQL, and away we go...right?

<img alt='' class='alignnone size-full wp-image-1372 ' src='/img/2016/10/img_58075f43d6c1d.png' />

Nope, same error, same stack dump. OK, let's call Microsoft back! Which brings us to where we are today. Their recommendation at this point is to drop and recreate the availability group. That seems like the sledgehammer approach but they say it's the only course of action at this point. 

We believe that we finally understand why this happened. We've been cleaning up individual logins on some of our SQL Servers (the ones that get created when you install SQL Server). That was what orphaned our AG. As soon as we removed the login that had created the AG, the owner principal got set to NULL and now we're in quite the pickle. We have been able to reproduce this with other logins on a test system. The bigger problem is that the AG ownership cannot be corrected without dropping/creating the AG. I know I've run into databases with NULL owners and never had a problem resetting ownership there. Those commands to reset the owner shouldn't stack dump, which sounds like a bug to me.


So for now this is what I know. If I find out any more information I will publish it here. I had a lot of discussions on Twitter over the last couple days regarding this and wanted to say thanks to all who offered input. Hopefully the next person who searches for availability group ownership issues won't come to the end of Google like I did.





