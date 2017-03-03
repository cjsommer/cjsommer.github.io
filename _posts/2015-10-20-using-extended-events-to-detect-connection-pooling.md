---
layout: post
title: Using Extended Events to Detect Connection Pooling
date: 2015-10-20 09:15
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
<hr />
What is connection pooling? Lets start with a definition from <a href="https://msdn.microsoft.com/en-us/library/bb399543(v=vs.110).aspx" target="_blank">MSDN</a>.

<em>Connecting to a data source can be time consuming. To minimize the cost of opening connections, ADO.NET uses an optimization technique called connection pooling, which minimizes the cost of repeatedly opening and closing connections. Connection pooling is handled differently for the .NET Framework data providers.</em>

Opening connections is a very expensive and time consuming operation. Connection pooling helps alleviate that by providing a reusable "pool" of connections that your application can share. Connection pooling is usually a good thing. Unfortunately I have found that it's sometimes not very well understood. This blog post starts with a short story...

I was working with my developers on a performance problem recently and I posed the question, "Is the application using connection pooling?", and the developers weren't really sure. So we did some reading about connection pooling on MSDN, and based on the connection string we thought we were but we wanted confirmation.

One way to validate connection pooling is from the application server. There are a number of <a href="https://msdn.microsoft.com/en-us/library/ms254503(v=vs.110).aspx" target="_blank">perfmon counters</a> available to monitor connection pooling from the application server side. My problem was that I didn't have access to the application servers. So as a DBA I went looking for a way to detect connection pooling from SQL Server, which was when I found <a href="http://blogs.msdn.com/b/sql_pfe_blog/archive/2013/10/08/connection-pooling-for-the-sql-server-dba.aspx" target="_blank">this article</a> that mentioned using Extended Events (near the bottom of the post).

Thank you to the <a href="http://blogs.msdn.com/b/sql_pfe_blog/archive/2013/10/08/connection-pooling-for-the-sql-server-dba.aspx" target="_blank">SQL Server Premier Field Engineering</a> blog for posting that. The is_cached event field is the one that identifies a pooled connection and it's just what I needed. 

This was one of the first times I have used Extended Events for something other than just tinkering around so I had to share. In the remainder of this post I will illustrate how to use Extended Events to detect connection pooling.

<hr />

<h3>First, I Needed a Way to Test Connection Pooling</h3>
Once again I reached for my good friend <a href="http://www.datamanipulation.net/SQLQueryStress/" target="_blank">SQLQueryStress</a>. SQLQueryStress is written in .NET and gives me the ability to enable or disable connection pooling in the options menu which is just what the doctor ordered.

<img class="alignnone size-full wp-image-1056 " src="/img/2015/10/img_561ee82ccbb9a.png" alt="" />

I setup SQLQueryStress to connect to my local database server and run a simple query against the AdventureWorks2012 database. It is set to run 10 iterations across 5 different threads. This should produce 50 individual queries.
<p id="YJiXxyu"><img class="alignnone size-full wp-image-1057 " src="/img/2015/10/img_561ee8adc70ab.png" alt="" /></p>

<hr />

<h3>Next, I needed my Extended Events Session</h3>
I am fairly new to using Extended Events so the first thing I did was fire up the wizard and see what I had to work with out of the box. Most of what I came up with was right in front of me as default templates and configurations in the wizard. Rather than post screenshots of me running through the wizard, I am just going to share the T-SQL script that I used to create the XE session. Once it has been created you can check it out in the SSMS GUI if you so desire. The main thing to understand about this session is that I am capturing the LOGIN event as it contains all of the information I need to detect connection pooling.

Note: If you want to use the script to create an XE session, make sure you change the output file path to a valid path on your server.
<pre class="theme:ssms2012 lang:tsql decode:true " title="XE Session Definition">
CREATE EVENT SESSION [ConnectionPooling] ON SERVER ADD EVENT sqlserver.connectivity_ring_buffer_recorded (
 ACTION(sqlserver.client_app_name, sqlserver.client_connection_id, sqlserver.client_hostname, sqlserver.context_info, 
  sqlserver.database_name, sqlserver.server_principal_name, sqlserver.session_id, sqlserver.sql_text)
 )
 ,ADD EVENT sqlserver.LOGIN (
 SET collect_options_text = (1) ACTION(sqlserver.client_app_name, sqlserver.client_connection_id, sqlserver.client_hostname, 
  sqlserver.context_info, sqlserver.database_name, sqlserver.server_instance_name, sqlserver.server_principal_name, 
  sqlserver.sql_text)
 ) ADD TARGET package0.event_file (SET filename = N'C:\SQL\MSSQL11.INST1\MSSQL\Log\ConnectionPooling.xel')
 WITH (
   MAX_MEMORY = 4096 KB
   ,EVENT_RETENTION_MODE = ALLOW_SINGLE_EVENT_LOSS
   ,MAX_DISPATCH_LATENCY = 30 SECONDS
   ,MAX_EVENT_SIZE = 0 KB
   ,MEMORY_PARTITION_MODE = NONE
   ,TRACK_CAUSALITY = ON
   ,STARTUP_STATE = OFF
   ) ;
</pre>

My test is setup. My XE session is ready to go. At this point I am ready to begin my test. 

<hr />

<h3>Test Method</h3>
<ol>
<li>
Start the XE session. Right click on the ConnectionPooling session in SSMS and select "Start Session".
<p><img alt='' class='alignnone size-full wp-image-1065 ' src='/img/2015/10/img_561eed80a4df6.png' /></p>
</li>
<li>
Start the SQLQueryStress test. Its as simple as hitting the big "GO" button.
<p><img class="alignnone size-full wp-image-1057 " src="/img/2015/10/img_561ee8adc70ab.png" alt="" /></p>
</li>
<li>
Stop the XE session. Right click on the ConnectionPooling session in SSMS and select "Stop Session"
<p><img alt='' class='alignnone size-full wp-image-1067 ' src='/img/2015/10/img_561eee22ad378.png' /></p>
</li>
<li>Analyze the results. 
<ol>
<li>Open the XEL file created by the ConnectionPooling XE session.</li>
<li>Add the columns client_host_name, client_app_name and is_cached to the display.</li>
<li>Group the data by client_host_name, client_app_name and is_cached.</li>
<li>Add an aggregation to make the counts stand out.</li>
</ol>
<p>It will be much clearer in the results posted below.
</li>
</ol>

<p>Based on my test I would expect a total of 50 queries from my SQLQueryStress test. 10 queries run across 5 threads = 50 queries run. Because of some other things SQLQueryStress does behind the scenes these numbers are not exact, but they are close to 50. What really stands out is when connection pooling is enabled or disabled for the following tests. You can definitely see connection pooling in action here.

<hr>
<h3>Test Results: Connection Pooling Disabled in SQLQueryStress</h3>
<img alt='' class='alignnone size-full wp-image-1088 ' src='/img/2015/10/img_561f0653f00cb.png' />

<ul>
<li>".Net SqlClient Data Provider" is the client_app_name for SQLQueryStress. </li>
<li>You can see that is_cached = false for all logins from SQLQueryStress, indicating that connection pooling is not enabled here. </li>
<li>Makes sense because that's how SQLQueryStress was configured for this test run.</li>
</ul>

<hr>
<h3>Test Results: Connection Pooling Enabled in SQLQueryStress</h3>
<img alt='' class='alignnone size-full wp-image-1089 ' src='/img/2015/10/img_561f06d038cc6.png' />

<ul>
<li>Once again ".Net SqlClient Data Provider" is the client_app_name for SQLQueryStress. </li>
<li>You can see in this case that is_cached = false for only 7 of the logins, and is_cached = true for 46.</li>
<li>The 7 logins where is_cached = false were not using pooled connections. These are the initial connections from SQLQueryStress.</li>
<li>The 46 logins where is_cached = true are using pooled connections. This is connection pooling in action.</li>
</ul>

<h3>Conclusion</h3>
Due to the cost of opening and closing connections, connection pooling is an important component when it comes to application performance. Extended Events give us all the tools we need as DBA's to see it in action. This is a very simple test and I highly recommend playing around with it a bit. I'm thinking this is just the tip of the iceberg with Extended Events for me.
