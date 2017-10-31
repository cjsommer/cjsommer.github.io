---
layout: post
title: Slow Running Query? Where do I Begin?
date: 2015-12-29 14:27
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
Maybe you're new to SQL Server or maybe you're just expanding into performance tuning a little more. Query tuning can be a very daunting task and sometimes it's hard to decide where to look first. In this post I am going to show you a couple of my go-to methods for troubleshooting slow performing queries in SQL Server. This is not a "ZOMG my SQL Server is slow" list. This post is more geared to when I have been tasked to dig into a very specific query. This isn't a be-all-end-all list, but its a good place to start and has served me well.

For the rest of this post I will be using the stored proc "spGetNames" as my problem query.

<hr />

<h2>Look at the execution plan</h2>
Analyzing an execution plan is an art form and I'm somewhere between a kindergartner with finger paints and Picasso (much closer to the finger paints). Sometimes the problem sticks out like a sore thumb and sometimes not, but I think an execution plan is a good place to start.

If you can run spGetNames safely against production, use SSMS and the "Include Actual Execution Plan" button. This will dump a graphical representation of the actual execution plan to SSMS and possibly help you determine where your query is running slow.

[img_slowquery]: /img/2015/12/slowquery_1.png
![My Slow Query][img_slowquery]

If you can't run spGetNames safely in production, get it from the plan cache. Thank you to Jonathan Kehayias (<a href="https://www.sqlskills.com/blogs/jonathan/" target="_blank">B</a> &#124; <a href="https://twitter.com/SQLPoolBoy" target="_blank">T</a>) over at SQLSkills for providing the basic query that I used. I just modified it to search for a specific query text. Jonathan's original query can be found <a href="https://www.sqlskills.com/blogs/jonathan/tuning-cost-threshold-for-parallelism-from-the-plan-cache/">here</a>. Below is my modified query. You just have to alter the WHERE clause to search for your query.

+EDIT: I'd like to thank Mike Fal (<a href="http://www.mikefal.net/" target="_blank">B</a> &#124; <a href="https://twitter.com/Mike_Fal" target="_blank">T</a>) for helping point out that the plans that come from the plan cache are estimated plans. Most of the time they will be the same as the actual plan, but there times where that may not be true, but it's a very important point to note.

```sql
SELECT  
     query_plan AS CompleteQueryPlan, 
     n.value('(@StatementText)[1]', 'VARCHAR(4000)') AS StatementText, 
     n.value('(@StatementOptmLevel)[1]', 'VARCHAR(25)') AS StatementOptimizationLevel, 
     n.value('(@StatementSubTreeCost)[1]', 'VARCHAR(128)') AS StatementSubTreeCost, 
     n.query('.') AS ParallelSubTreeXML,  
     ecp.usecounts, 
     ecp.size_in_bytes 
FROM sys.dm_exec_cached_plans AS ecp 
CROSS APPLY sys.dm_exec_query_plan(plan_handle) AS eqp 
CROSS APPLY query_plan.nodes('/ShowPlanXML/BatchSequence/Batch/Statements/StmtSimple') AS qn(n) 
WHERE n.value('(@StatementText)[1]', 'VARCHAR(4000)') 
    LIKE '%spGetNames%' -- Alter the search text to find your stored proc here
```

And here is the output. If you click on the Complete Query Plan XML field you will see the graphical query plan that came from the plan cache.
<img class="alignnone size-full wp-image-1178 " src="/img/2015/12/slowquery_2.png" alt="" />

<hr />

<h2>Statistics parser</h2>
TIME and IO statistics can tell you a lot about spGetNames. I find these stats very useful as they display the IO, CPU and TIME statistics for a given query. All you have to do is add a couple lines of TSQL before and after spGetNames and it will dump the stats for you.

<img class="alignnone size-full wp-image-1188 " src="/img/2015/12/slowquery_3.png" alt="" />

But that's not the coolest part. Richie Rump (<a href="http://www.jorriss.net/" target="_blank">B</a> &#124; <a href="https://twitter.com/Jorriss" target="_blank">T</a>) has built an online tool that will parse your statistics output and give you a friendlier view of the data. The URL for this awesome tool is <a href="http://statisticsparser.com/index.html#" target="_blank">StatisticsParser</a>. All you have to do is to paste the statistics output from the SSMS message box and hit the parse button and whala...a nice display of the query statistics. This isn't a great example because the query I used is pretty simple, but hopefully you get the idea.

<img class="alignnone size-full wp-image-1191 " src="/img/2015/12/img_5682c8128e7e8.png" alt="" />

<hr />

<h2>SQL Trace/Extended Events</h2>
I am just going to mention SQL Trace/XE here for completeness sake as I don't think it's a great tool for analyzing performance issues, especially when you are already focused in on a specific query. I think it's a useful tool if you are still trying to identify the problem query, but as far as digging into a specific query or procedure I don't really get much value out of it. The execution plan and statistics give me the same information without having to run an intrusive trace or XE session. SQL Trace and Extended events are always the last tool I reach for.

Disclaimer: SQL Profiler/Trace and Extended Events can have a significant performance impact on your database server. I always refine my trace to be as narrow as possible using a DEV or TEST system before I even think about running it on a PROD server. ALWAYS!

<hr />

<h3>Conclusion</h3>
Just like Prego spaghetti sauce, it's in there. SQL Server contains all of the information you need to dig into a poorly performing query, you just need to know where to find it. Execution plans and statistics are usually the first couple tools in my tool belt that I reach for. Hopefully this was helpful to some. Thanks for reading!

Aside: I want to mention that this information is useful all the way up through SQL 2014. SQL 2016 is a game changer as far as performance tuning because of a new feature called Query Store. Here's a snippet from <a href="https://msdn.microsoft.com/en-us/library/dn817826.aspx" target="_blank">MSDN</a>.

<em>The SQL Server Query Store feature provides DBAs with insight on query plan choice and performance. It simplifies performance troubleshooting by enabling you to quickly find performance differences caused by changes in query plans. The feature automatically captures a history of queries, plans, and runtime statistics, and retains these for your review. It separates data by time windows, allowing you to see database usage patterns and understand when query plan changes happened on the server. </em>

I haven't had the time to try it out yet, but I am dying to dig into it.
