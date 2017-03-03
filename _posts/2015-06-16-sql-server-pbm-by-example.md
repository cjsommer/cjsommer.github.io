---
layout: post
title: SQL Server Policy Based Management by Example
date: 2015-06-16 08:30
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
What steps do you go through to validate SQL Server configurations after a server build? Even a single server can take some time and is also prone to human error. Have you ever needed to validate a configuration setting across all of the SQL Servers in your environment? If your environment is big enough, doing this manually isn't even a realistic option. Policy Based Management excels at these things. Introduced in SQL Server 2008, it can definitely help DBA's manage their ever growing environments.

Every time I use Policy Based Management (PBM) I say to myself, "Self, you have GOT to do more of this"! Put it together with Central Management Server and you have a great platform for ensuring that your database standards are being followed across all of your SQL Servers.

This blog post will give a very high level overview of the PBM components in an effort to spark some interest in a less heralded, but very useful feature.

<div style="clear:both" /><hr />
<h3>What are all of the Components of PBM?</h3>
<a href="/img/2015/06/PBM_components.png"><img class=" size-full wp-image-714 alignright" src="/img/2015/06/PBM_components.png" alt="PBM_components" width="321" height="236" /></a>
You will find PBM and its 3 main components listed under the Management tab of SSMS. The 3 components are Policies, Conditions and Facets. 

SQL Server ships with a standard set of policies, but they aren't installed for you by default. To import the standard policies, right click on the Policy node in SSMS and import them. The default policies are stored in the following location on my SQL 2012 install. 

<code>C:\Program Files\Microsoft SQL Server\110\Tools\Policies\DatabaseEngine\1033</code>

The path should be similar no matter what version of SQL Server you are using. The rest of this blog post I hope to explain how all of the pieces fit together. It's not as complicated as it may look.

<div style="clear:both" /><hr />
<h3>Policy</h3>
The Policy is the highest level component. It is a container that contains the Check Condition, the Targets, the Evaluation Mode, and the Server Restriction. 

The example I am using is the "Backup and Data File Location" policy. It checks to ensure that the default backup and data file locations are configured to use different drives.

<a href="/img/2015/06/PBM_PolicyExampleExample.png"><img src="/img/2015/06/PBM_PolicyExampleExample.png" alt="PBM_PolicyExampleExample" width="716" height="617" class="alignnone size-full wp-image-729" /></a>

<h3>Check Condition</h3>
<a href="/img/2015/06/PBM_ConditionExample1.png"><img src="/img/2015/06/PBM_ConditionExample1.png" alt="PBM_ConditionExample" width="530" height="208" class="alignright size-full wp-image-742" /></a>
The check condition is a logical comparison that will return either true or false. In this case, it will return TRUE is backup and data files are on separate drives and return FALSE if they are on the same drive. 

The objects and property values that you can test for are known as facets. The facets you have to work with come with every SQL Server installation (2008 and above). Each new version of SQL Server adds new facets for new features, and can also add to existing facets. PBM was introduced in SQL 2008, but you can use it against SQL 2000 and 2005 as well.

<h3>Targets (Condition)</h3>
<a href="/img/2015/06/PBM_Targets.png"><img src="/img/2015/06/PBM_Targets.png" alt="PBM_Targets" width="427" height="214" class="alignleft size-full wp-image-740" /></a>
The targets in a policy define what databases you want to run the policy against. A target is just the result of another condition. There are a number of commonly used targets included, or you can create your own condition for your desired target.

<h3>Evaluation Mode</h3>
You have the option to run the policy "On Demand" or "On Schedule". On Demand allows you to run the policy any time you want to manually. On Schedule allows you to schedule the policy check using a SQL Agent job. For my purposes in this blog post I'll just be running this check manually.

<h3>Server Restrictions (Condition)</h3>
<a href="/img/2015/06/PBM_serverrestriction.png"><img src="/img/2015/06/PBM_serverrestriction.png" alt="PBM_serverrestriction" width="424" height="181" class="alignright size-full wp-image-739" /></a>
Server restrictions are another condition that defines which version of SQL Server you want to run this policy against. Once again when you install the default policies that come with SQL Server it will include a number of pre-configured server restrictions, but you can always add your own. 

In the Backup and Data File Location policy there is no server restriction. 

<h3>Example: Running against my local SQL Server Instance</h3>
<a href="/img/2015/06/PBM_Evaluate.png"><img src="/img/2015/06/PBM_Evaluate.png" alt="PBM_Evaluate" width="327" height="180" class="alignright size-full wp-image-750" /></a>
To run the policy manually, right click on it and select Evaluate

And Here are the results
<a href="/img/2015/06/PBM_Results.png"><img src="/img/2015/06/PBM_Results.png" alt="PBM_Results" width="955" height="502" class="alignnone size-full wp-image-751" /></a>

Yeah, I know. My data and backup files are on the same location. Definitely not a best practice and the policy check caught me. Luckily it's just on my laptop where I do most of my demos and blog posts, and I don't have multiple drive letters available to me so I knew this would be the result. If this were one of my production SQL Servers I would definitely be taking some action to remedy this. 

So there you have it. I successfully ran a policy check against my local server instance. This is a very basic example of what you can do with Policy Based Management. If you have not played with PBM I recommend trying it out. There is a lot more to it than I have described here, but I am hoping this post sparked your interest into giving it a try. It's a really cool and powerful feature to ensure your databases are configured the way you want them to be.
