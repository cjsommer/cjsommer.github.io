---
layout: post
title: Approaching Big Problems
date: 2025-10-25T01:00:00
author: cjsommer@gmail.com
comments: true
tags: ["Professional"]
---

Being laid off last month put me back on the job hunt, a journey I haven't been on in a long while. I've already been through a couple rounds of interviews with a handful of different companies. But if I'm honest with myself I don't think they went all that well because I was not well prepared. In my interviews a common theme arose; being presented with a hypothetical situation and being asked how I would attack it.

For me, explaining how to attack a problem is a lot different than actually sitting down at the keyboard and attacking it. I'm tactile. I like to feel things. I like to witness cause and effect. What happens when I turn this knob or flip this switch? And I use those observations to guide me.

I felt the answers I gave were over generalized. I just didn't know how to organize my thoughts and present them very well in an interview setting.

So this blog post is mostly for me. I'm going to try to capture my process in writing and hopefully help myself out in future interviews. And who knows, maybe someone else will find it useful.

## The Problem
The database server for the core business application has been under duress for the last few weeks. Query latency is up across the board. Applications are experiencing timeouts during peak hours. Reporting replicas are falling behind and providing less value to the business. And we just signed on a whale of a customer that is planning to go live in 6 months.


### Possible Causes
There could be a number of causes for this type of behavior, especially if it just started happening recently.
- Business
    - Did we just add new customers that are contributing to the load?
- Application
    - Any app changes with new database work?
    - And not just major app changes. I've seen seemingly innocuous changes have significant performance impact.
- Database
    - New columns and maybe forgot to update an index?
    - New tables and forgot indexes?
    - New queries?

For the purposes of this exercise we've ruled out any major application changes. The business has been growing but nothing out of the ordinary over the past few months. So this is just mainly organic growth and we've hit a bit of a tipping point on the database infrastructure.

## High Level Approach
- Short term - What can we do right now to stop the bleeding?
- Medium term - What can we do to help stabilize for the next 12 months?
- Long term - What do we need to do to ensure stability for the long term?

## Short Term (first aid)
Stop the bleeding!

- Look at the database
    - Are we vacuuming? Does any tables need tuning?
    - Are we analyzing? Bad statistics are bad for query performance.
- Look at the database infrastructure
    - If we are experiencing excessive CPU load then bumping the instance class might give us some breathing room
    - If disk I/O is getting pegged (IOPS or Throughput) then increase the IO subsystem

## Medium Term (optimization)
Just because the bleeding has stopped doesn't mean we're done. We still have a major customer to onboard in the next 6 months. We need to continue to optimize.

- Optimize frequently run queries
    - Look for queries with high execution counts. Even simple queries can ripple through the system if they're executed frequently enough.
    - Eliminate key lookups and ensure your indexes are completely covering
- Optimize less frequently run but poor performing queries
    - Look for queries with longer durations
    - Can we eliminate joins?
    - Is indexing appropriate?
- Are we using batching where appropriate?
    - Look for RBAR (row by agonizing row) processes
    - Batch processing offer performance gains over singleton operations
- Remove unnecessary queries
    - The best query is one you don't run.
- Are we keeping our transactional data lean and mean?
    - Do we have data lifecycle policies in place?

## Long Term (architecture)
The bleeding has stopped. We've optimized a number of components in our existing system and gotten past the 1 year mark. DBRE's are no longer getting woken up at 2:00 AM and the business is happy. But we want our company to be prepared for the future so what's next? I think the next step is talking about architecture of the system as a whole.

Relational databases scale vertically, but vertical scalability has a limit. We already get horizontal scalability for reads, but it's not a native feature for writes on most RDBMS platforms. There are some middleware components that offer to do it (Vitess for MySQL & Elastic Database in Azure) but there will be implications on the app side. TL;DR it's gonna take some work on the app side to implement a sharding solution.

I think this discussion can be delayed based on where your business is in its journey, but I also believe you should having that conversation well before your system needs it.
