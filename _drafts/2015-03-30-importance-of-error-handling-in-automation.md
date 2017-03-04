---
layout: post
title: Importance of Error Handling in Automation
date: 2015-03-30 11:53
author: cjsommer@gmail.com
comments: true
categories: [Uncategorized]
---
My definition of automation is simple. Automation is having computers perform repetitive tasks for us.

There are a number of reasons why we automate processes, but generally speaking automation brings stability to a potentially complex process. Computers are good at performing repetitive tasks, people not so much. Automation could be a simple script or it could be a complex program. In the SQL Server world it could be a TSQL script, a PowerShell script, or an SSIS package. It could be a script that I run myself, or it could be a script that is run on a regular basis using a job scheduler. Bottom line, if it helps me perform a set of steps to complete a task, I consider it automation.

OK, so we got that out of the way. We get it. Automation is having computers perform a repetitive task for us. So what about the title of the post? What is this about error handling and why is it so important?

Automation is easy when everything just works. We press the button and the script roars to life. Files are moving between servers, databases are being updated in the cloud, results are being rolled up, reports are being generated, all while I sit back and enjoy a delicious cup of coffee from my french press. But that is not the world we live in. The computer world is a harsh one! Servers go down, the network is slow and firewalls block our every move! Those are just a few specific examples of what we might need to deal with, but you get the picture. With a manual process we get to decide what action to take next when things don't work quite as planned. With automation, we have to program that into our scripts. Anyone can write a script for the perfect world, but the best automated solutions are written to handle the real world.

<h2>Understand the Methodologies</h2>
Understand the error handling methodologies at your disposal, and that may be different depending on which technology you are using for your process. For example:
<ol>
	<li>For TSQL we have TRY/CATCH blocks</li>
	<li>PowerShell we have Try/Catch/Finally blocks</li>
	<li>In SSIS you can create Event Handlers to deal with errors</li>
</ol>
<h2>Handle the Errors</h2>
Cannot tell you how to handle each error because it is specific to the process you are trying to automate. But there are only a few options when you do run into an error in your script.
<ol>
	<li>Determine that the error wasn't important and continue on.</li>
	<li>Stop the script and notify the operator of the failure.</li>
	<li>Retry the operation. (with a max number of retries)</li>
</ol>
<h2>Test the Error Conditions</h2>
Test your error handling routines for both pass and fail conditions.
<h2>Test again. Error handling is that important.</h2>
