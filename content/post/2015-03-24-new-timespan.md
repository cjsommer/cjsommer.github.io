---
layout: post
title: Using New-TimeSpan to convert friendly time
date: 2015-03-24T18:16:00
author: cjsommer@gmail.com
comments: true
tags: ["Powershell"]
---
The question was this:

<a href="/img/2015/03/New-TimeSpan.jpg"><img class="alignnone size-full wp-image-24" src="/img/2015/03/New-TimeSpan.jpg" alt="New-TimeSpan" width="265" height="133" /></a>

This is just a quick/fast snippet in response to that question, but I can think of a ton of enhancements that could made. Hopefully it will be useful as a starting point for someone to customize for their own use.

```powershell
function Convert-FriendlyTime
{
    param (
        [string]$FriendlyTime 
    )
    
    $Converted = $FriendlyTime.Split(" ")
    
    switch ($Converted[1])
    {
    "days"    { $parms = @{"days"=$Converted[0]}; break ;}
    "hours"   { $parms = @{"hours"=$Converted[0]}; break ;}
    "minutes" { $parms = @{"minutes"=$Converted[0]}; break ;}
    "seconds" { $parms = @{"seconds"=$Converted[0]}; break ;}
    "default" { $parms = $null; break ;}
    }
    if ($parms) {
        New-TimeSpan @parms
    } else {
        Throw "Invalid unit of time '$FriendlyTime'. Please use days, hours, minutes or seconds."
    }
}

Convert-FriendlyTime -FriendlyTime "2 days"
Convert-FriendlyTime -FriendlyTime "68 hours"
Convert-FriendlyTime -FriendlyTime "68 InvalidUnits"
```
<h3>PowerShell is fun!</h3>
