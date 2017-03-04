---
layout: post
title: TSQL2SDAY 70: Strategies for managing an enterprise
date: 2015-09-08 14:11
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
<a href="/img/2015/05/TSQLTuesday.jpg"><img src="/img/2015/05/TSQLTuesday.jpg" alt="TSQLTuesday" width="150" height="150" class="alignright size-full wp-image-504" /></a>
Thanks to MidnightDBA (<a href="http://www.midnightdba.com/" target="_blank">b</a>/<a href="https://twitter.com/MidnightDBA" target="_blank">t</a>) for hosting this month's #tsql2sday. 

<a href="http://www.midnightdba.com/Jen/2015/09/time-for-t-sql-tuesday-70/" target="_blank">Strategies for managing an enterprise.</a>

What do you think of when you hear the word enterprise? I used to think mainframes and big iron. I would think of sprawling server rooms with dozens of server racks. I used to think monstrous storage arrays and backup tape libraries as far as the eye could see. I used to think of a large employee base. For me the word enterprise used to correlate to size, but I don't think that any more...

I think being an enterprise has more to do with the quality than quantity or size. When I think of an enterprise I think of stability. I think of standards. I think of change control processes and procedures. I think of scalable solutions. I think of supportability.

As a DBA this brings to mind a couple thoughts. 

<h3>Service Level Agreements (RPO & RTO) Based on Business Need</h3>
Service Level Agreements (SLA's) for Recovery Point Objective and Recovery Time Objectives (RPO and RTO) should be determined and documented for every application. RPO is the amount of data the business is willing to lose in the event of a database failure. RTO is amount of time that the business has determined that the database can be down in the event of a failure. RPO and RTO are actually determined by the needs of the business, not by the DBA's. The DBA is responsible for architecting a database solution that meets the business needs.

Can you tell me what your SLA's are for all of your databases? If it's not documented and reviewed on a fairly regular basis, there is a good chance that there are differing expectations depending on who you talk to. During a crisis is not the time to find that out!

<h3>Central Management</h3>
Central Management Server is the first thing that comes to mind, but this could also be a homegrown solution written in PowerShell or something like that if you so desired. The function of a Central Management Server is a single place to manage all of the servers you are responsible for. That could be 2 servers, it could be 2000. Regardless, it's nice to have all of your servers listed and available in one place. 

Central Management Server is nice because it comes with SQL Server and is really easy to setup. There are a couple nice things you get when you setup a CMS.
<ol>
<li>First and foremost it gives you a consolidated listing of all your servers. </li>
<li>It is also a great platform to run queries across multiple servers at the same time. This is useful for pulling information from your SQL Server inventory, or even in deploying maintenance scripts or updates to them.</li>
<li>CMS can also be used as a management point for Policy Based Management. Policy checks can even be automated using CMS and the <a href="http://epmframework.codeplex.com/">EPM Framework</a>. This is nice to help keep new SQL builds in check.</li>
</ol>

<h3>Database Maintenance</h3>
Performing proper database maintenance is a core competency of being a DBA. Backups, statistics, index rebuilds, consistency checks; a DBA craves all these things! Database maintenance should be driven by the requirements of the business (see SLA's above), not by the feelings of the DBA.

Maintenance Plans are The Devil and should be avoided at all costs! They offer limited visibility into what is going on behind the scenes and are a pain in the ass to troubleshoot. When I think of an enterprise level maintenance strategy I think of a scripted solution. Something that can be deployed from a CMS or a PowerShell script. Something like <a href="https://ola.hallengren.com/">OLA Hallengren's</a> solution, the <a href="http://minionware.net/">Minion scripts</a> or even a homegrown solution. Regardless, I think whatever standard you pick should be consistent across the enterprise. This standardization is a godsend when you are troubleshooting failed backup jobs at 3:00 AM.

<hr>
Those are just a couple thoughts and I have more, but it's really a big subject. I truly believe that being an enterprise has much more to do with the quality of your environment that it does the size. Spending a little more time up front to <strong>architect a solution</strong> rather than accomplish a task goes a long way to building your enterprise!
