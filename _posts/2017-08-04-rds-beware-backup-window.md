---
layout: post
title: Beware the automated backup window when running native SQL Server backups in RDS
date: 2017-08-04 00:00
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
# Beware the automated backup window when running native SQL Server backups in RDS

If you've ever fired up an RDS instance you know you can set an automated backup window for your instance. During this window Amazon will kick off an automated snapshot of your RDS instance each day. Set it and forget it. Kinda nice. Backups are good, right? 

Another nice feature if you are using SQL Server on RDS is that it supports native SQL Server backup and restore using S3 buckets. This can be helpful for getting data into and out of RDS, or in my case we use it for migrating databases between RDS instances. The following link describes in detail how to configure this feature for your RDS instances as it's not enabled by default.

<a href="http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/SQLServer.Procedural.Importing.html">http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/SQLServer.Procedural.Importing.html</a>

Once you have enabled native SQL backup/restore, using them is very straight forward. Amazon provides a couple wrapper stored procedures that allow us to perform these functions quite easily. 

```sql
-- Backup the AdventureWorks database to S3
use msdb ;

exec msdb.dbo.rds_backup_database
    @source_db_name='AdventureWorks',
    @s3_arn_to_backup_to='arn:aws:s3:::sqlserver-backups/AdventureWorks_full.bak',
    @overwrite_S3_backup_file=1;

-- Restore the AdventureWorks database from the S3 bucket
use msdb ;
 
exec msdb.dbo.rds_restore_database
    @restore_db_name='AdventureWorks',
    @s3_arn_to_restore_from='arn:aws:s3:::sqlserver-backups/AdventureWorks_full.bak';
```

When I run either the backup or restore command it actually adds a request to a job queue, and RDS processes that queue asynchronously behind the scenes. You can check the status of the queue using the following stored proc.

```sql
use msdb ;

exec msdb.dbo.rds_task_status ;
```
<img src="/img/2017/08/rds_task_status.png" alt="rds_task_status" align="left">

As you can see, it's pretty simple to do native backup and restore in RDS, but there is one gottcha I have run into that I didn't see in the documentation. If you start a native SQL backup/restore operation, and you run into the automated RDS backup window, RDS will kill your native SQL backup/restore. The rds_task_status keeps a very detailed output and it's pretty easy to see what happened, but it can be a pain in the neck if you start a backup only to find it killed before completion.

```
 [2017-08-04 23:46:23.690] Task execution has started. 
 [2017-08-04 23:55:48.560] 5 percent processed. 
 [2017-08-04 23:56:56.220] AdventureWorks_full.bak: Completed processing 5% of S3 chunks. 
 [2017-08-05 00:05:35.700] 10 percent processed. 
 [2017-08-05 00:07:33.177] AdventureWorks_full.bak: Completed processing 10% of S3 chunks. 
 [2017-08-05 00:09:08.180] Aborted the task because of a task failure or an overlap with your preferred backup window for RDS automated backup. 
 [2017-08-05 00:09:08.230] AdventureWorks_full.bak: Aborting S3 upload, waiting for S3 workers to clean up and exit 
 [2017-08-05 00:09:09.240] AdventureWorks_full.bak: S3 processing has been aborted 
 [2017-08-05 00:09:37.950] Write on "A000000-6CB1-4416-8C61-0673E45C9B50" failed: 995(The I/O operation has been aborted because of either a thread exit or an application request.) 
 [2017-08-05 00:09:37.957] A nonrecoverable I/O error occurred on file "A000000-6CB1-4416-8C61-0673E45C9B50:" 995(The I/O operation has been aborted because of either a thread exit or an application request.). 
 [2017-08-05 00:09:37.960] BACKUP DATABASE is terminating abnormally.
```

One option to work around this issue is to disable the automated backups, but please note that this will delete all of snapshots for that instance. Probably not something you want to do. You never need backups until you do. 

The only other way around this issue is to never run native SQL backup/restore commands around your scheduled RDS backup window. Maybe you need to move your automated backup window. Maybe you need to time your native backup/restore. Maybe both.

Not sure I'm happy about this behavior, but it's kind of out of our control. If nothing else I hope I raised some awareness and gave everyone something to consider. Thanks for reading!