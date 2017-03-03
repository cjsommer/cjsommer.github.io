---
layout: post
title: Convert User Friendly Retention to DateTime value with PowerShell
date: 2016-10-25 12:00
author: cjsommer@gmail.com
comments: true
categories: [PowerShell, SQL Server]
---
<img src="/img/2015/03/sql-posh-300x250.png" alt="sql-posh" width="300" height="250" class="alignright size-medium wp-image-179" />

I think the title is fairly descriptive so let me put a little context around it for you. In my SQL Server environment we backup our databases to local disk. Unfortunately we don't have unlimited storage for backups, which means we have to delete the old backups on a regular basis. A very typical practice in the SQL DBA world.

I was writing a new cmdlet for <a href="https://dbatools.io/" target="_blank">dbatools.io</a> (Remove-DbaBackup to be released in October) and needed to decide how I wanted users to provide the retention for their SQL backups. I've seen a very wide range of backup retention requirements in my career (hours, days, weeks, months, years). I could have coded to the least common denominator of hours and greatly simplified my work, but I wanted to find a more flexible and elegant solution. The last thing I wanted was for people to have to fire up calc.exe, or worse yet having to take off their shoes to figure out how many hours were in a month. This is the function I came up with to create a more user friendly experience with my cmdlet.
 
<pre class="lang:ps decode:true " title="Convert-UserFriendlyRetentionToDatetime" >function Convert-UserFriendlyRetentionToDatetime
{
    [cmdletbinding()]
    param (
        [string]$UserFriendlyRetention
    )

    &lt;# 
    Convert a user friendly retention value into a datetime.
    The last character of the string will indicate units (validated) 
    Valid units are: (h = hours, d = days, w = weeks, m = months)

    The preceeding characters are the value and must be an integer (validated)
    
    Examples: 
        '48h' = 48 hours
        '7d' = 7 days
        '4w' = 4 weeks
        '1m' = 1 month
    #&gt;

    [int]$Length = ($UserFriendlyRetention).Length
    $Value = ($UserFriendlyRetention).Substring(0,$Length-1)
    $Units = ($UserFriendlyRetention).Substring($Length-1,1)   

    # Validate that $Units is an accepted unit of measure
    if ( $Units -notin @('h','d','w','m') ){
        throw "RetentionPeriod '$UserFriendlyRetention' units invalid! See Get-Help for correct formatting and examples."
    }

    # Validate that $Value is an INT
    if ( ![int]::TryParse($Value,[ref]"") ) {
        throw "RetentionPeriod '$UserFriendlyRetention' format invalid! See Get-Help for correct formatting and examples."
    }

    switch ($Units)
    {
        'h' { $UnitString = 'Hours'; [datetime]$ReturnDatetime = (Get-Date).AddHours(-$Value)  }
        'd' { $UnitString = 'Days';  [datetime]$ReturnDatetime = (Get-Date).AddDays(-$Value)   }
        'w' { $UnitString = 'Weeks'; [datetime]$ReturnDatetime = (Get-Date).AddDays(-$Value*7) }
        'm' { $UnitString = 'Months';[datetime]$ReturnDatetime = (Get-Date).AddMonths(-$Value) }
    }
    Write-Verbose "Retention set to '$Value' $UnitString. Retention date/time '$ReturnDatetime'"
    $ReturnDatetime
}</pre> 

This function accepts the -UserFriendlyRetention parameter and returns the datetime value calculated from that retention. 

<h1>How to use it?</h1>

You pass in a numeric value along with 1 of 4 different units of measure (h, d, w, m). Rather than try to elaborate further how about some examples with screenshots? I think it's pretty clear once you see how it works. 

<h3>Ex. 24h = 24 hours</h3>
<img alt='' class='alignnone size-full wp-image-1428 ' src='/img/2016/10/img_580f5ea7e2f1c.png' />

<h3>Ex. 7d = 7 days</h3>
<img alt='' class='alignnone size-full wp-image-1429 ' src='/img/2016/10/img_580f5ed294423.png' />

<h3>Ex. 4w = 4 weeks</h3>
<img alt='' class='alignnone size-full wp-image-1430 ' src='/img/2016/10/img_580f5ef646410.png' />

<h3>Ex. 6m = 6 months</h3>
<img alt='' class='alignnone size-full wp-image-1431 ' src='/img/2016/10/img_580f5f149db62.png' />

Hopefully you get the picture. It's pretty straight forward and I think it accomplishes my goal of not having to fire up calc.exe (or take off your shoes *pew*) just to figure out your retention date. I'm sure there are plenty of other uses for this function and it could definitely be customized for other units of measure if you so desired. Feel free to use and enjoy, and let me know if you have any questions. 

<strong>Happy scripting, everyone!</strong>

