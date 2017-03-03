---
layout: post
title: Get-DiskSpace
date: 2015-03-24 17:40
author: cjsommer@gmail.com
comments: true
categories: [PowerShell]
---
My cheap and dirty PowerShell script for dumping the disk space usage for a Windows machine. It was originally some snippets I found on the Internet, which I threw into a script and made it my own.

It uses a simple WMI call to return a set of objects for you to use as you see fit. Most of the time I just pipe it to Format-Table to see the output, but it could be used for other things. Throw in a pipe to Where-Object and you could use it to monitor low disk space. Feel free to bind, spindle or mutilate as you see fit.

It has been tested on Windows 2003 and above. If you're still using Windows 2000, I'm sorry. Good luck and have fun.
```PowerShell
 .Notes
 NAME: Get-DiskSpace.ps1
 AUTHOR: Chris Sommer
 Version: 1.0
 CREATED: 6/6/2012
 LASTEDIT:
 6/6/2012 - 1.0 - Initial Release

 .Synopsis
 Get disk space listing.

 .Description
 This script gets a disk space listing including mount points. Can be run locally or remotely using Invoke-Command.

 .Example
 Run on the local computer
 .\Get-DiskSpace.ps1

 .Example
 Run against a remote computer. Must have WinRM enabled on the remote computer.
 Invoke-Command -Computer MyServerName .\Get-DiskSpace.ps1
#&gt;

$unit = "MB"
$measure = "1$unit"

Get-WmiObject -query "
select SystemName, Name, DriveType, FileSystem, FreeSpace, Capacity, Label
  from Win32_Volume
 where DriveType = 2 or DriveType = 3" `
| select SystemName `
        , Name `
        , @{Label="SizeIn$unit";Expression={"{0:n2}" -f($_.Capacity/$measure)}} `
        , @{Label="FreeIn$unit";Expression={"{0:n2}" -f($_.freespace/$measure)}} `
        , @{Label="PercentFree";Expression={"{0:n2}" -f(($_.freespace / $_.Capacity) * 100)}} `
        ,  Label
```

Sample output from my local machine (machine names and user names blanked out)

<a href="{{ site.baseurl }}/images/2015/03/Get-DiskSpace.ps1_.png">
<img class="alignnone size-full wp-image-16" src="{{ site.baseurl }}/images/2015/03/Get-DiskSpace.ps1_.png" alt="Get-DiskSpace.ps1" width="640" height="106" />
</a>
