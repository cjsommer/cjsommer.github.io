---
layout: post
title: Pratical Use of a SQL Server Database Snapshot
date: 2015-09-23 09:15
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
I was doing a database code deployment the other night and the first step in the release plan was "Take a full database backup in case we have to roll back the deployment". The database in question was about 100 GB in size, so not huge, but not small either. On our current hardware a full backup takes anywhere from 15-20 minutes on average. That's 15-20 minutes longer than it needed to be because I knew there was a better way to offer that same rollback protection. The database snapshot.

Database snapshots are a feature that was introduced with SQL Server 2005 that allows you to create a transactionally consistent picture of your database at the moment the snapshot was taken. Unfortunately it's an Enterprise Edition feature only so if you are on anything other than Enterprise Edition you wont be able to use snapshots. I still think it's a feature worth knowing about.

<h3>So is a snapshot like a backup?</h3>
Well, kinda but not really. It's like a backup in the sense that it can be used as a point in time to restore back to, but it can not be used for media recovery. This is because of how a snapshot works. The snapshot relies on the original data file(s) being intact to be usable for recovery. When you create a database snapshot you define a snapshot file for each data file in your database. Each snapshot file starts out empty, but as changes are made to the original database, the original data pages are written to this snapshot file. The preserves the original state of the data pages, and is used when you restore from a snapshot. Microsoft does a very good job explaining it on their MSDN page for <a href="https://msdn.microsoft.com/en-us/library/ms175158(v=sql.120).aspx" target="_blank">Database Snapshots (SQL Server)</a>.

<h3>So what's so cool about snapshots?</h3>
I like snapshots for rollback protection in deployments for a couple reasons. 
<ol>
<li>They are fast! Creating a snapshot of our 100 GB database takes less than a second. A full backup as I said earlier is a 20 minute operation.</li>
<li>Restoring from a snapshot is typically a very quick operation as well, depending on the delta of the original database after the snapshot was taken. For the purposes of a code release, this delta is typically very small and the restore from the snapshot takes less than a minute.</li>
<li>They are simple. The TSQL syntax for managing snapshots is really easy to understand.
</ol>

Snapshots are super easy to manage and they have very little performance impact on the underlying database. And the best part is you always have your standard database backups to fall back on in case you need to. You have fulls, diffs and tlogs and could always be used as a second form of protection, but there's no reason to take another full backup during a code deployment. Why waste 20 minutes waiting for a backup before you begin your code deployment when you have snapshots available to you?

<h3>Creating a Snapshot</h3>
Below is an example script for creating a database snapshot for the AdventureWorks2012 database. You will see that a snapshot file is defined for each data file in the database. If there were multiple data files there would be multiple snapshot files. It's pretty straight forward.

<pre class="theme:ssms2012 lang:tsql decode:true " title="Create Snapshot" >
CREATE DATABASE AdventureWorks2012_SnapShot
    ON
    (
        NAME = 'AdventureWorks2012_Data',
        FILENAME ='C:\SQL\MSSQL11.INST1\MSSQL\DATA\AdventureWorks2012_Data.ss'
    ) 
    AS SNAPSHOT OF AdventureWorks2012 ;
</pre> 

You will now find your snapshot under Database Snapshots in SSMS
<a href="/img/2015/09/DatabaseSnapshots.png"><img src="/img/2015/09/DatabaseSnapshots.png" alt="DatabaseSnapshots" width="302" height="59" class="alignnone size-full wp-image-933" /></a>

<h3>Restoring from a Snapshot</h3>
Restoring from a database snapshot is also a simple operation. SQL Server handles all of the dirty work and figures out what pages need to be written back to the data files from the snapshot files all behind the scenes. The syntax couldn't be much simpler really.
<pre class="theme:ssms2012 lang:tsql decode:true " title="Restore from Snapshot" >
RESTORE DATABASE AdventureWorks2012 
    FROM DATABASE_SNAPSHOT = 'AdventureWorks2012_SnapShot' ;
</pre>

<h3>Dropping a snapshot</h3>
And dropping a database snapshot is just like dropping a database. Once again, very simple syntax here. 
<pre class="theme:ssms2012 lang:tsql decode:true " title="Drop Snapshot" >
DROP DATABASE [AdventureWorks2012_SnapShot] ;
</pre> 

<h3>What else can I use a snapshot for?</h3>
Taken directly from the MSDN link posted above, here are some other use cases for snapshots.
<ul>
	<li>Snapshots can be used for reporting purposes.</li>

	<li>Maintaining historical data for report generation.</li>

	<li>Using a mirror database that you are maintaining for availability purposes to offload reporting.</li>

	<li>Safeguarding data against administrative error.</li>

	<li>In the event of a user error on a source database, you can revert the source database to the state it was in when a given database snapshot was created. Data loss is confined to updates to the database since the snapshot's creation.</li>

</ul>

<h3>Conclusion</h3>
I was really surprised that as part of this database deployment nobody knew about database snapshots. They had just accepted the fact that a database deployment included a full backup. This was completely unnecessary and we were able to shorten the deployment time by 20 minutes by using database snapshots instead. Hopefully this article enlightens someone and maybe cuts a few minutes out of your deployment.
