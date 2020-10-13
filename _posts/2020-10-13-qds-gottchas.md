---
layout: post
title: QDS Forced Plans - Gotchas and Limitations
share-img: http://www.cjsommer.com/img/2020/10/qds_tech_debt.jpg
meta-description: Released with SQL Server 2016, QDS was a game changer for performance tuning. Forced plans are an amazing tool for the DBA toolbox, but they do come with a few things you should know about.
date: 2020-10-13 00:00
author: cjsommer@gmail.com
comments: true
image: /img/2020/10/qds_tech_debt.jpg
categories: [SQL Server]
---
<!-- Image and URL references used in this post -->
[img_qds_force_plan]: /img/2020/10/qds_force_plan.png
[img_qds_tech_debt]: /img/2020/10/qds_tech_debt.jpg
[img_qds_purge]: /img/2020/10/qds_purge.png
[img_qds_no_soup]: /img/2020/10/qds_no_soup_for_you.png

[url_qds_ms]: https://docs.microsoft.com/en-us/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store

<!-- Content -->

<img src="/img/2020/10/qds_tech_debt.jpg" alt="QDS Forced Plans" align="right">

Released with SQL Server 2016, Query Data Store was a game changer for performance tuning. Right from Microsoft's website: 

_The SQL Server Query Store feature provides you with insight on query plan choice and performance. It simplifies performance troubleshooting by helping you quickly find performance differences caused by query plan changes. Query Store automatically captures a history of queries, plans, and runtime statistics, and retains these for your review. It separates data by time windows so you can see database usage patterns and understand when query plan changes happened on the server._

[Official QDS documentation from microsoft.com][url_qds_ms]

One of QDS signature abilities allows you to force a plan for a specific query. With query plan and performance history right at your fingertips it became quite easy to see a plan regression, as well as fix it! Forcing a plan in QDS allows you to push a button and tell SQL Server to always use the "good" plan for your query. In a world where we seem to be in a never ending battle with parameter sniffing and plan stability it seemed to good to be true. This might just have been the silver bullet we were looking for! 

![QDS Image][img_qds_force_plan]

...but I don't believe in silver bullets (or werewolves for that matter). While forced plans are amazing and have gotten me out of a number of sticky situations, they do come with their own set of gotchas and limitations.

---

## GOTCHA #1 - Availability Group read only replicas? No pinned plans for you!
<img src="/img/2020/10/qds_no_soup_for_you.png" alt="No forced plans for you!" align="right">

You're already using Availability Groups and you decide you want to offload some of your read workload off to a readable secondary. Sounds like a great solution trying to scale out a bit. If you're leveraging forced plans you're kinda out of luck. Forced plans are only honored on the primary replica. SQL Server will not use forced plans on secondary replicas. There currently is no way around this, although there are plenty of bitches, gripes, and complaints about it in the SQL Server community. 

---

## GOTCHA #2 - The Footgun
Once in a while SQL Server gives you the ability to shoot yourself in the face. QDS forced plans are no exception with the "Purge Query Data" button that you see in the GUI (as well as the matching T-SQL command). 

![QDS Purge][img_qds_purge]

The purge button does not discriminate. When it say purge, it means purge all the things, which includes any forced plans you have. If you're leveraging forced plans to fine tune your workload this could spell certain disaster on a production system. You can imagine that this risk goes up on how much you're actually using it.

---

## GOTCHA #3 - Why won't my query use my new index?
Once you have a forced plan in place, you may not benefit from new indexes in the future. Lets take this example:

You get a ticket asking you to look at the performance for a query. Upon investigation it's pretty clear that it needs a better index. Maybe the table started off small and grew to the point of where performance became an issue, which is not an uncommon problem to have. So you slap your new index in place but you can't for the life of you get the query to use it. You try everything under the sun and it just won't work. Of course you have a 2:00 AM epiphany - QDS forced plans! Sure enough you un-force the plan and your index is finally being used. 

This is a bit contrived example but it's not far from what could happen. How much time did you waste? How many other queries are sitting out there like this?

---

## GOTCHA #4 - Keeping forced plans in sync across all environments
So you've pinned a plan in production to "fix" a bad query plan. This becomes a very large part of how that query performs from that point forward. If you do any sort of performance testing in lower environment you'd probably want to keep the forced plans there in sync with what's in production to ensure your performance profiles match. So how do you keep your forced plans in sync in lower environments, or do you even attempt to do it at all? 

Every query in QDS has a query ID, and these query ID's are scoped to the database where the query is running. This means that query ID's from production are not the same as the query ID's in your lower environments for a given query. 

If you're lucky enough to be using stored procedures then this won't be awful to keep in sync. It is a little more work and a little more difficult to match query text using ORM's, but it can still be done. The biggest problem with this is the human factor. People are going to be the biggest reason that forced plans are not kept consistent across environments. This becomes a manual process, and if you've done any reading on my blog you'll know how I feel about manual processes.

If you regularly refresh your lower environments from production backups then this won't be a big deal because the forced plans come with the database. But I know some of us don't do that. 

---

# QDS Forced Plans - It's not all bad
As a matter of fact I think it's an amazing feature to have in a pinch. It's saved me a number of times when a query has gone south and I needed to fix the performance immediately. I guess the key phrase there is "in a pinch" because forced plans come with a cost, and I consider that cost technical debt. 

Until some of the limitations and gotchas of forced plans have been addressed I feel it's in our best interest as DBA's to work on more permanent solutions toward query performance and plan stability. And if you've been doing this job long enough then you understand that's a blog post for another time.
