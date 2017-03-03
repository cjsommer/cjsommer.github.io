---
layout: post
title: Gotcha! PowerShell Array Initialization, Difference between PowerShell 2.0 and 4.0
date: 2015-03-24 18:24
author: cjsommer@gmail.com
comments: true
categories: [PowerShell]
---
I like to initialize my arrays by strongly typing my variable as an array, and setting it to null. PowerShell doesn't require strongly typed variables, so why would I want to do this?

The primary reason is because it allows me to deal with an expected object type later in the code. For arrays it gives me access to a count property, which I don't get with strings, int's or other non-collection type objects.

Much of the scripting I do involves running queries against SQL Server. A query may return multiple rows, it may return 1 row, or it may even return 0 rows. Having access to the count property is a major part of the flow control I use in a lot of the scripts I write, which once again is why I like strongly typing my arrays.

There's a really sneaky gotcha in PowerShell 2 because it handled null arrays differently than PowerShell 4 does, which is the main point of this post. In PowerShell 2.0 a null array returns a count of NULL. This breaks any conditional checks that reference the count property.
<pre class="theme:powershell-ise toolbar:1 scroll:true lang:ps decode:true nums:false ">C:\Users\BIGRED-7&gt; $psversiontable

Name                           Value
----                           -----
CLRVersion                     2.0.50727.5485
BuildVersion                   6.1.7601.17514
PSVersion                      2.0
WSManStackVersion              2.0
PSCompatibleVersions           {1.0, 2.0}
SerializationVersion           1.1.0.1
PSRemotingProtocolVersion      2.1


C:\Users\BIGRED-7&gt; [array]$myvar1 = $null
C:\Users\BIGRED-7&gt; $myvar1.count
C:\Users\BIGRED-7&gt;
</pre>
This involved creative workarounds for conditional statements in a lot of scripts I wrote back in the PowerShell 2.0 days. If anyone is interested I can dig some of the workarounds up, but I highly recommend just upgrading to PowerShell 4.0 and not looking back. Reason being, in PowerShell 4.0 a null array returns a count of 0 as I would expect.
<pre class="theme:powershell-ise toolbar:1 scroll:true lang:ps decode:true nums:false ">C:\Users\BIGRED-7&gt; $psversiontable

Name                           Value
----                           -----
PSVersion                      4.0
WSManStackVersion              3.0
SerializationVersion           1.1.0.1
CLRVersion                     4.0.30319.18063
BuildVersion                   6.3.9600.16406
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0}
PSRemotingProtocolVersion      2.2


C:\Users\BIGRED-7&gt; [array]$myvar2 = $null
C:\Users\BIGRED-7&gt; $myvar2.count
0
C:\Users\BIGRED-7&gt;
</pre>
This got me a number of times back in the PowerShell 2.0 days. I highly recommend that if you are still running PowerShell 2.0 you upgrade to 4.0 (5.0 is on the near horizon as of the writing of this blog post). Hope this helps someone out somewhere.
