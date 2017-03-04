---
layout: post
title: Configure SQL Server Agent using SMO
date: 2015-03-24 18:05
author: cjsommer@gmail.com
comments: true
categories: [PowerShell, SQL Server]
---
Saw this on twitter and thought I would throw it up on the blog. Easier than responding on twitter.

<a href="/img/2015/03/SQLAgentConfiguration.png"><img class="alignnone size-full wp-image-25" src="/img/2015/03/SQLAgentConfiguration.png" alt="SQLAgentConfiguration" width="273" height="131" /></a>

```powershell
[System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SqlServer.Smo") | Out-Null;

$SQLServerInstance = "MSSQLSERVER"

$srv = New-Object "Microsoft.SqlServer.Management.Smo.Server" $SQLServerInstance
$srv.JobServer.MaximumHistoryRows = 1002
$srv.JobServer.Alter()
```

You can manipulate most if not all SQL Server configurations through SMO. This simple post has inspired me a bit. I think it would be useful to show how I came to find the information I needed to modify this setting. The cool thing about PowerShell is that it's in there. All the tools you need already exist. Look back for an expansion of this post in the near future.
