---
layout: post
title: T-SQL Tuesday #83: How are we still dealing with the same problems?
date: 2016-10-11 16:26
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
<img src="/img/2015/05/TSQLTuesday.jpg" alt="TSQLTuesday" width="150" height="150" class="alignright size-full wp-image-504" />

#TSQL2SDAY is a monthly blog party hosted by a different blogger each month. This blog party was started by Adam Machanic (<a href="http://sqlblog.com/blogs/adam_machanic/default.aspx" target="_blank">blog</a>|<a href="https://twitter.com/AdamMachanic" target="_blank">twitter</a>). You can take part by posting your own participating post that fits the topic of the month and follows the requirements below. Additionally, if you are interested in hosting a future T-SQL Tuesday, contact Adam Machanic on his blog.

<a href="http://am2.co/2016/10/t-sql-tuesday-83/" target="_blank"><img alt='' class='alignnone size-full wp-image-1339 ' src='/img/2016/10/img_57fbc653ac62d.png' /></a>

<hr div="clear">
<h3>So why are we still dealing with the same problems after all these years?</h3>
We're still dealing with the same problems because we're dealing with the problems in the same way. 

I think it can be cultural and can propagate from the senior level DBA's right on down to the new hires. Sometimes it's just lack of knowledge or understanding. Sometimes it's just pure laziness to not want to do a deep dive and find a better solution to a recurring problem.

Here is a pretty extreme example but I think it portrays all of these.

You're a new DBA and it's your first day on the job at Company X. You're manning the problem queue when you get a storage alert on a backup drive on one of your servers. OMG this is so awesome! My first real issue! I'm so gonna own this! So you go see your mentor, a senior level DBA who's been with the company for 20 years. He looks at the issue with you briefly and says, "Yeah, just clean up some of the old backup files. That'll make the alert go away." So you clean up those backup files, the alert goes away and you feel awesome. You just fixed an issue and you will fix many more just like it in the years to come.

Fast forward 2 years. You're no longer the new guy on the block and another new DBA starts with your team. Your manager assigns you as the mentor to the new guy when low and behold he gets a storage alert. Of course you're an expert by this time after having dealt with thousands of storage alerts in your 2 years as a DBA so you graciously show him the ropes. Before you know it he's deleting backup files like a pro. Mission accomplished!

Fast forward another 2 years. They decide to hire another DBA, but this time they go with a more experienced one. Of course she starts on the problem queue trying to get a feel for the environment when a storage alert comes in to her. She has a lot of experience so she knows how to dig a little bit and discovers that the maintenance jobs are just not configured to purge old backups. Duh! She adds the cleanup step to the backup jobs and you never see another storage alert again.

Once again, a very extreme example, but I think it illustrates how a culture can just accept a problem as "normal" when they really shouldn't. The senior level DBA was just being lazy. Don't really know why but he never dug into the problem or he also would have noticed that it was an easy permanent fix. The first junior DBA just didn't know any better. Can't really fault him for that. Same for the second junior DBA as well since he was taught by the first junior DBA. The final more experienced DBA finally dug into it because she brought in other experiences and knowledge.

<img alt='' class='alignright size-full wp-image-1343 ' src='/img/2016/10/img_57fd40a0ec86b.png' />

I know I got to rambling a bit so what's my point? Stop making the alerts go away and start double-tapping your problems. If you have a recurring issue it means that there is more to be discovered before you can truly fix the problem. Dig into it. Your teammates will thank you.





