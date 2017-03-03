---
layout: post
title: Free SQL Server Performance Testing Utilities
date: 2015-06-02 08:15
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
Did that say free? Why yes, yes it did! But don't let the price tag fool ya, these are some pretty nice little utilities.
<a href="/img/2015/05/free-tools-tweet.jpg"><img class="alignright size-full wp-image-644" src="/img/2015/05/free-tools-tweet.jpg" alt="free-tools-tweet" width="266" height="141" /></a>
Another blog post inspired by a question on Twitter.

I went to SQL Saturday #383 a few weeks ago and noticed a handful of performance testing utilities being mentioned or used during many of the presentations. I have seen them all mentioned or used in demos before, but I have very little experience using them myself. I thought it would be nice to gather them all in one place for future reference, and I can definitely foresee a more in depth review of each down the road.

Below you will find the name of the utility, a brief description of what it is primarily used for (taken from the creator's website), and the URL of the creator's website.

<span style="color: #ff0000;"><strong>Warning: These utilities are made to put a significant load on your database server so use at your own risk. Make sure your resume is up to date if you choose to run them in production!</strong></span>

In all seriousness, understand what these tools do before you attempt to run them in any of your environments. If you run these utilities against a server that uses a shared resource like a SAN, they can affect other hosts using that SAN. If you run them against a VM, they can impact othere guests on the same host. This advice really goes for any tool you might use, but a couple of these have the ability to significantly impact performance on a live system.
<hr />

<h3>SQLIO</h3>
Description: SQLIO is a tool provided by Microsoft which can also be used to determine the I/O capacity of a given configuration.

Website: <a href="http://www.microsoft.com/en-us/download/details.aspx?id=20163" target="_blank">SQLIO Download</a>

<hr />

<h3>DiskSpd</h3>
Description: DiskSpd is the successor to SQLIO. DISKSPD is a storage load generator / performance test tool from the Windows/Windows Server and Cloud Server Infrastructure Engineering teams.

Website: <a href="https://github.com/microsoft/diskspd" target="_blank">DiskSpd on Github</a>

<hr />

<h3>DVDStore</h3>
Description: The Dell DVD Store is an open source simulation of an online ecommerce site with implementations in Microsoft SQL Server, Oracle, MySQL and PostgreSQL along with driver programs and web applications.

Website: <a href="http://linux.dell.com/dvdstore/" target="_blank">http://linux.dell.com/dvdstore/</a>

<hr />

<h3>HammerDB</h3>
Description: HammerDB is an open source database load testing and benchmarking tool for Oracle, SQL Server, TimesTen, PostgreSQL, Greenplum, Postgres Plus Advanced Server, MySQL, Redis and Trafodion SQL on Hadoop.

Website: <a href="http://www.hammerdb.com/" target="_blank">http://www.hammerdb.com/</a>

<hr />

<h3>SQLQueryStress</h3>
Description: SQLQueryStress by Adam Machanic is a free tool for SQL Server programmers. It is designed to assist with performance stress testing of T-SQL queries and routines. The tool automatically collects metrics to help you determine whether your queries will perform under load, and what kind of resource strain they put on your server.

Website: <a href="http://www.datamanipulation.net/sqlquerystress/" target="_blank">http://www.datamanipulation.net/sqlquerystress/</a>

<hr />

There you have it, a list of some really neat performance test utilities for your SQL Servers. Stay tuned for a deeper dive into each of them over the next few months.
