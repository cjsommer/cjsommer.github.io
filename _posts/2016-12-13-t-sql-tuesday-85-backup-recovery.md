---
layout: post
title: T-SQL Tuesday #85 - Backup & Recovery
date: 2016-12-13 10:00
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
<img src="http://www.cjsommer.com/wp-content/uploads/2015/05/TSQLTuesday.jpg" alt="" width="150" height="150" class="alignright size-full wp-image-504" />
<p>Shout out to Kenneth Fisher (<a href="https://sqlstudies.com/">b</a>|<a href="https://twitter.com/sqlstudent144">t</a>) for hosting this month&#39;s #tsql2sday. This month is about backup and recovery and here is a snippet from the invite:</p>
<p><em>Backups are one of the most common things DBAs discuss, and they are at once one of the simplest and most complicated parts of our whole job. So let’s hear it for backup and recovery!</em></p>
<p>You can see the whole post on his <a href="https://sqlstudies.com/2016/12/06/4169/">blog</a>.</p>
<h1>SQL Server Recovery Models</h1>
<p>Yeah, lets go with that. There are 3 different recovery models in SQL Server, Full, Simple, and Bulk Logged. Understanding the intricacies of each is important in determining which is best for you. In this post I plan to share what I feel are some of the important points of each.
<h3>Full Recovery Mode</h3>
<p>Full recovery is the most granular recovery model and gives us the ability to restore our databases to any point in time. This is usually my default recovery mode.</p>
<ul>
<li>Point in time recovery.</li>
<li>All transactions are written to the transaction log and persist there until the next log truncation. Log truncation happens after a transaction log backup.</li>
<li>Transaction log backups required! Fail to do so and your transaction log drive will eventually run out of space.</li>
<li>Required for Availability Groups, Database Mirroring and Log Shipping.</li>
</ul>
<h3>Simple Recovery Mode</h3>
<p>One of the biggest misconceptions I have seen about recovery models has been surrounding simple mode. People think that by using simple mode you&#39;re getting a performance boost because transactions are no longer logged. This is absolutely not true. Transactions are still logged, they&#39;re just not persisted or backed up for recovery purposes.</p>
<ul>
<li>No point in time restores. We can only restore back to our full and differential backups.</li>
<li>Simple mode does not mean no transaction log activity. Transactions are still logged in simple mode. They are just not persisted in the transaction log after they complete (committed or rolled back).</li>
<li>You can still run out of transaction log space while in simple mode if you have a large enough transaction.</li>
<li>Log backups are not required as log truncation happens after a checkpoint.</li>
</ul>
<h3>Bulk Recovery Mode</h3>
<p>I&#39;ve only seen bulk logged mode one time and it was for an ETL process that needed the extra boost in performance. It was also a rerunnable process so if we had to go back and recover the database we would have had to rerun the ETL process over again. Again, it&#39;s the only time I&#39;ve seen it used and it was for a very specific use case.</p>
<ul>
<li>Bulk logged is typically a temporary recovery mode to be used only during bulk operations.</li>
<li>Similar to full recovery mode, except for bulk operations. Bulk operations are minimally logged (bcp, BULK INSERT, and INSERT... SELECT).</li>
<li>Point in time recovery is not supported.</li>
<li>Same as full recovery, transaction log backups required! Fail to do so and your transaction log drive will eventually run out of space.</li>
</ul>
<h3>Summary</h3>

<p>Those are some of what I feel are the more important points of each recovery model. It&#39;s not an all inclusive list by any means and you can find the full details on <a href="https://msdn.microsoft.com/en-us/library/ms189275.aspx">MSDN</a> if you are so inclined.</p>

<p><p>Generally speaking, my recommendation is to always use the full recovery model unless you can explain exactly why you don&#39;t need to. Understanding the finer details goes a long way to making the correct decisions for your environment.</p>

