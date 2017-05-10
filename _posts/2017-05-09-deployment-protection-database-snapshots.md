---
layout: post
title: T-SQL Tuesday 90 - Shipping Database Changes
date: 2017-05-09 20:00
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
T-SQL Tuesday is a monthly blog party for the SQL Server community (or Microsoft Data Platform community. Although it’s called T-SQL Tuesday, it’s not limited to SQL Server database engine only). It is the brainchild of Adam Machanic [blog](http://sqlblog.com/blogs/adam_machanic/) &#124; [twitter](https://twitter.com/AdamMachanic).

<img src="/img/TSQLTuesday.jpg" alt="tsql2sday" align="right">

This month's tsql2sday is being hosted by James Anderson [blog](http://thedatabaseavenger.com/2017/05/t-sql-tuesday-shipping-database-changes/) &#124;[twitter](https://twitter.com/DatabaseAvenger) and this month's theme is "Shipping Database Changes". For this T-SQL Tuesday, I’d like to hear about your thoughts or experiences with database deployments.

---

Well, my thoughts for today's T-SQL Tuesday are about covering your a$$ because database deployments don't always go as planned. There's a number of reasons why a code deployment may have to be rolled back, and sometimes that includes the database. It's a road I've been down more than once (sadly). What I've learned over the years is that it's always good to have a solid rollback plan. A way to get the database back to the pre-deployment state. Sure you could always restore from backups, but if you have a large database that can take quite a while. For my first line of defense I prefer to use database snapshots.

Database snapshots are a feature that was introduced with SQL Server 2005 that allows you to create a transactionally consistent picture of your database at the moment the snapshot was taken. Unfortunately it's an Enterprise Edition feature only so if you are on anything other than Enterprise Edition you wont be able to use snapshots. I still think it's a feature worth knowing about.

### So a snapshot is like a backup?
Well, kinda but not really. It's like a backup in the sense that it can be used as a point in time to restore back to, but it can not be used for media recovery. This is because of how a snapshot works. The snapshot relies on the original data file(s) being intact to be usable for recovery. When you create a database snapshot you define a snapshot file for each data file in your database. Each snapshot file starts out empty, but as changes are made to the original database, the original data pages are written to this snapshot file. The preserves the original state of the data pages, and is used when you restore from a snapshot. Microsoft does a very good job explaining it on their MSDN page for <a href="https://msdn.microsoft.com/en-us/library/ms175158(v=sql.120).aspx" target="_blank">Database Snapshots (SQL Server)</a>.

### So what's so cool about snapshots?
I like snapshots for rollback protection in deployments for a couple reasons. 

* They are fast! Creating a snapshot of our 100 GB database takes less than a second. A full backup as I said earlier is a 20 minute operation.
* Restoring from a snapshot is typically a very quick operation as well, depending on the delta of the original database after the snapshot was taken. For the purposes of a code release, this delta is typically very small and the restore from the snapshot takes less than a minute.
* They are simple. The TSQL syntax for managing snapshots is really easy to understand. No finding back files and running RESTORE commands in the heat of battle.


Snapshots are super easy to manage and they have very little performance impact on the underlying database. And the best part is you always have your standard database backups to fall back on in case you need to. You have fulls, diffs and tlogs and could always be used as a second form of protection, but there's no reason to take another full backup during a code deployment. Why waste 20 minutes waiting for a backup before you begin your code deployment when you have snapshots available to you?

### Creating a Snapshot
Below is an example script for creating a database snapshot for the AdventureWorks2012 database. You will see that a snapshot file is defined for each data file in the database. If there were multiple data files there would be multiple snapshot files. It's pretty straight forward.

```sql
CREATE DATABASE AdventureWorks2012_SnapShot
    ON
    (
        NAME = 'AdventureWorks2012_Data',
        FILENAME ='C:\SQL\MSSQL11.INST1\MSSQL\DATA\AdventureWorks2012_Data.ss'
    ) 
    AS SNAPSHOT OF AdventureWorks2012 ;
```

You will now find your snapshot under Database Snapshots in SSMS
<a href="/img/2015/09/DatabaseSnapshots.png"><img src="/img/2015/09/DatabaseSnapshots.png" alt="DatabaseSnapshots" width="302" height="59" class="alignnone size-full wp-image-933" /></a>

### Restoring from a Snapshot
Restoring from a database snapshot is also a simple operation. SQL Server handles all of the dirty work and figures out what pages need to be written back to the data files from the snapshot files all behind the scenes. The syntax couldn't be much simpler really.

```sql
RESTORE DATABASE AdventureWorks2012 
    FROM DATABASE_SNAPSHOT = 'AdventureWorks2012_SnapShot' ;
```

### Dropping a snapshot
And dropping a database snapshot is just like dropping a database. Once again, very simple syntax here. 

```sql
DROP DATABASE [AdventureWorks2012_SnapShot] ;
```

### What else can I use a snapshot for?
Taken directly from the MSDN link posted above, here are some other use cases for snapshots.
* Snapshots can be used for reporting purposes.
* >Maintaining historical data for report generation.
* Using a mirror database that you are maintaining for availability purposes to offload reporting.
* Safeguarding data against administrative error.
* In the event of a user error on a source database, you can revert the source database to the state it was in when a given database snapshot was created. Data loss is confined to updates to the database since the snapshot's creation.


### Conclusion
I was really surprised that as part of this database deployment nobody knew about database snapshots. They had just accepted the fact that a database deployment included a full backup. This was completely unnecessary and we were able to shorten the deployment time by 20 minutes by using database snapshots instead. Hopefully this article enlightens someone and maybe cuts a few minutes out of your deployment.
