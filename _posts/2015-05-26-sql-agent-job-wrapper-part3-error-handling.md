---
layout: post
title: SQL Agent Job Wrapper Part 3 - Handing the Errors in the Wrapper Script
date: 2015-05-26 08:15
author: cjsommer@gmail.com
comments: true
categories: [PowerShell, SQL Server]
---
This is the 3rd installment in a small series of blog posts on how to create a PowerShell wrapper for running SQL Server Agent Jobs. Here are the links to the 2 previous posts and I recommend reading them because all of the posts build on the previous one.
 
<a href="http://www.cjsommer.com/creating-a-sql-agent-job-wrapper-with-powershell-and-smo/">Creating a SQL Agent Job Wrapper with PowerShell and SMO - Part 1</a>
<a href="http://www.cjsommer.com/sql-agent-job-wrapper-part2-error-generation/">SQL Agent Job Wrapper Part 2 – Adding Error Generation to the Cmdlet</a>

<hr>
<h5>SQL Agent Job Wrapper Part 3 - Handing the Errors in the Wrapper Script</h5>
Now that we have the Start-SQLAgentJob cmdlet tested and working like we need, we have to build a wrapper script. The cmdlet could be called directly from the command line, but if you are going to run SQL Agent Jobs through any sort of a scheduler you will want to write a wrapper script. That is the purpose of the Test-Wrapper.ps1 script from my previous posts. The Test-Wrapper.ps1 script will be responsible for calling the Start-SQLAgentJob cmdlet and handling any errors it throws.
<hr> 
<h5>Start-SQLAgentJob.ps1</h5>
So just a little review. Below is the Start-SQLAgentJob cmdlet. It does all of the heavy lifting in this whole process and is responsible for throwing errors when appropriate. All of the error generation functionality was tested as part of last week's blog post. Here is the cmdlet code so you don't have to go back to last week's post to find it.

```powershell
function Start-SQLAgentJob
{
    <#
     .Notes
     NAME: Start-SQLAgentJob
     AUTHOR: Chris Sommer
     Version: 1.0
     CREATED: 2015-05-09

     .Synopsis
     Start a SQL Server Agent job.

     .Description
     Start a SQL Agent job and wait for its completion. This function relies on the SQL Agent to be up and running.

     .Parameter SQLServer
     SQL Server Name

     .Parameter JobName
     SQL Agent job name

     .Example
     Start-SQLAgentJob -SQLServer "localhost" -JobName "TestJob"
    #>
    [CmdletBinding()]
    param (
        [Parameter(Mandatory=$true)][string]$SQLServer ,
        [Parameter(Mandatory=$true)][string]$JobName
    )
    
    # Load the SQLPS module
    Push-Location; Import-Module SQLPS -DisableNameChecking; Pop-Location

    # ConnectionContext.Connect tests the connection to the SQLServer. This will throw an error if connection fails.
    $ServerObj = New-Object Microsoft.SqlServer.Management.Smo.Server($SQLServer)
    $ServerObj.ConnectionContext.Connect()

    # New check to ensure the JobName exists on this SQL Server
    if ( ($ServerObj.JobServer.Jobs | Where-Object {$_.Name -eq $JobName} | Select-Object -ExpandProperty Name) -ne $JobName ) {
        throw ("Job $JobName does not exist")
    }

    $JobObj = $ServerObj.JobServer.Jobs | Where-Object {$_.Name -eq $JobName}
    $JobObj.Refresh()

    # If the job is and enabled and not currently executing start it
    if ($JobObj.IsEnabled -and $JobObj.CurrentRunStatus -ne "Executing") {
        $JobObj.Start()
    }

    # Wait until the job completes. Check every second.
    do {
        Start-Sleep -Seconds 1
        # You have to run the refresh method to reread the status
        $JobObj.Refresh()
    } While ($JobObj.CurrentRunStatus -eq "Executing")

    # Get the run duration by adding all of the step durations
    $RunDuration = 0
    foreach($JobStep in $JobObj.JobSteps)     {
        $RunDuration += $JobStep.LastRunDuration
    }

    # If the job succeeded return the job object, otherwise throw an error.
    if ($JobObj.LastRunOutcome -eq "Succeeded") {
        $JobObj | select Name,CurrentRunStatus,LastRunOutcome,LastRunDate,@{Name="LastRunDurationSeconds";Expression={$RunDuration}}
    } else {
        $JobResult = $JobObj.LastRunOutcome
        throw ("Job '$JobName' LastRunOutcome = '$JobResult'")
    }
}
```

<hr>
<h5>Test-Wrapper.ps1</h5>
The Test-Wrapper.ps1 script is what calls the cmdlet and is responsible for the actual error handling. It traps any errors thrown by the Start-SQLAgentJob cmdlet and takes appropriate action. In this case it prints the error out to the screen and exits with a return code of 1. There is a lot more you can do with a try/catch block, but for the purpose of this post I decided to keep it simple. Here is the actual Test-Wrapper.ps1 script.

```powershell
Set-StrictMode -Version 2.0
$error.Clear()
$ErrorActionPreference = "Stop"

try {
    # Setup pathing and environment based on the script location
    $Invocation = (Get-Variable MyInvocation -Scope 0).Value
    $ScriptLocation = Split-Path $Invocation.MyCommand.Path
 
    # Load the Start-SQLAgentJob cmdlet
    . "$ScriptLocation\Start-SQLAgentJob.ps1"
 
    # Set a couple variables for testing and call the cmdlet
    $SQLServer = "localhost\inst1"
    $JobName = "TestJob"
    Start-SQLAgentJob -SQLServer $SQLServer -JobName $JobName
} catch {
    $error[0]
    throw $error[0]
    # Exit with error code of 1 on any failure
    $host.SetShouldExit(1) 
}
```

This version is quite a bit different than the simple Test-Wrapper script I have used in the previous 2 posts. I'll try to highlight the main differences here.
<ol>
<li>$error.Clear clears the error buffer. I like to do this at the beginning of all of my scripts to clear the error buffer. This ensures that I don't go chasing old errors in case of a script failure.</li>
<li>$ErrorActionPreference = "Stop". This will force any errors generated in the script to become terminating errors. Not all errors in PowerShell are terminating and because of my coding style I prefer to set this as my ErrorActionPreference.</li>
<li>All of the work is in a try/catch block. This is a great mechanism for trapping errors in a PowerShell script. Any terminating errors that happen in the try block with fall into the catch block. This catch block is very simple and will just throw the error and exit the script.</li>
<li>$host.SetShouldExit(1) - This will send a return code of '1' back to the calling program. This is the mechanism that I use to communicate back to the external scheduler that my script has failed. There may be other ways to do this, but this way has always worked for me so I continue to use it.</li>
</ol>
<hr>
I know that my cmdlet throws all of the errors that I expect because I tested it last week. So I am just going to show 1 example of using Test-Wrapper.ps1 that will intentionally throw an error.

Below are the results of a successful SQL Agent job, and a failed SQL Agent job. In the failed job you will see that the error message describes exactly what went wrong and because of the SetShouldExit it can be trapped by an external scheduler to indicate a failure.
<h5>Result from a Successful SQL Agent job</h5>
<a href="/img/2015/05/SQLAgent_SuccessfulJob.jpg"><img src="/img/2015/05/SQLAgent_SuccessfulJob.jpg" alt="SQLAgent_SuccessfulJob" width="649" height="193" class="alignnone size-full wp-image-584" /></a>
<h5>Result from a Failed SQL Agent job</h5>
<a href="/img/2015/05/SQLAgent_Wrapper_FailedJob.png"><img src="/img/2015/05/SQLAgent_Wrapper_FailedJob.png" alt="SQLAgent_Wrapper_FailedJob" width="922" height="248" class="alignnone size-full wp-image-610" /></a>
<h5>Conclusion</h5>
Wow this turned into a 3 part series, way more than I might have thought, but I wanted to break it down into smaller more digestible bites. I am hoping that I accomplished that. With PowerShell there are many ways to tackle a single problem. If you have another method that you'd like to share please feel free to add a comment below. I love questioning my methods and learning from others. As always, Happy Scripting!
