---
layout: post
title: SQL Agent Job Wrapper Part 2 - Adding Error Generation to the Cmdlet
date: 2015-05-20 08:15
author: cjsommer@gmail.com
comments: true
categories: [PowerShell, SQL Server]
---
Last week I posted <a href="http://www.cjsommer.com/creating-a-sql-agent-job-wrapper-with-powershell-and-smo/">Creating a SQL Agent Job Wrapper with PowerShell and SMO</a>. In that post I created a couple PowerShell scripts that run a SQL Agent job and wait for it to complete before exiting. That process could be called from the command line, or even from a 3rd party job scheduler if you were so inclined. I recommend checking it out before you continue, because this is a continuation of that post. 

<h5>SQL Agent Job Wrapper Part 2 - Adding Error Generation to the Cmdlet</h5>
When it comes to automation, error handling is one of the most important parts of building a truly robust solution. One of the most challenging things is trying to figure out how people are going to try and break your stuff. What are the possible failure points? 

For the SQL Agent job wrapper, all of the error generation is built into the cmdlet. When the cmdlet encounters an error it has to communicate that back up the stack to the calling script. This is also called "throwing" an error. For the Start-SQLAgentJob cmdlet I can think of 3 main error conditions that it might have to throw.  

Condition #1 - Invalid or unavailable SQL Server. What if my SQL Server doesn't exist or isn't running?  
Condition #2 - Invalid Job Name. What if the JobName I passed into the cmdlet does not exist?
Condition #3 - The job failed to run successfully.

If any of these conditions is met I will expect the cmdlet to throw an error. The original version of the cmdlet would only throw an error if it tried to connect to an invalid SQL Server. An invalid Job Name or job failure would not throw an error, so I had to add that functionality. Here is the new version of the cmdlet with the additional checks to validate the Job Name and the Last Run Status after the job completes. If any of those error conditions are met, the cmdlet will explicitly throw an error.
 
<pre class="lang:ps decode:true " title="Start-SQLAgentJob.ps1" >
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
</pre> 

<hr>
<h5>Successful Job Run</h5>
This is what a successful run of the cmdlet looks like. No errors are encountered and the Job object is returned to the calling script. The test script is pretty much the same test script that I used last week, with the addition of the ErrorActionPreference. ErrorActionPreference set to "stop" will ensure that the cmdlet halts on any error. I will explain that variable a bit more in Part 3 of this series.
<pre class="lang:ps decode:true " title="Successful SQL Agent Job Run" >
# Setup pathing and environment based on the script location
$Invocation = (Get-Variable MyInvocation -Scope 0).Value
$ScriptLocation = Split-Path $Invocation.MyCommand.Path
 
# Load the Start-SQLAgentJob cmdlet
. "$ScriptLocation\Start-SQLAgentJob.ps1"

# Set a couple variables for testing and call the cmdlet
$SQLServer = "localhost\inst1"
$JobName = "TestJob"

$ErrorActionPreference = "Stop"
Start-SQLAgentJob -SQLServer $SQLServer -JobName $JobName
</pre>
<a href="/img/2015/05/SQLAgent_SuccessfulJob.jpg"><img src="/img/2015/05/SQLAgent_SuccessfulJob.jpg" alt="SQLAgent_SuccessfulJob" width="649" height="193" class="alignnone size-full wp-image-584" /></a>

Below this point are the tests to validate each of the error conditions that I outlined above. 

<h5>Condition #1 - Invalid or unavailable SQL Server</h5>
<pre class="lang:ps decode:true " title="Bad SQLServer parameter" >
# Setup pathing and environment based on the script location
$Invocation = (Get-Variable MyInvocation -Scope 0).Value
$ScriptLocation = Split-Path $Invocation.MyCommand.Path
 
# Load the Start-SQLAgentJob cmdlet
. "$ScriptLocation\Start-SQLAgentJob.ps1"

# Set a couple variables for testing and call the cmdlet
$SQLServer = "localhost\badserver"
$JobName = "TestJob"

$ErrorActionPreference = "Stop"
Start-SQLAgentJob -SQLServer $SQLServer -JobName $JobName
</pre>
<a href="/img/2015/05/SQLAgent_BadSQLServer.png"><img src="/img/2015/05/SQLAgent_BadSQLServer.png" alt="SQLAgent_BadSQLServer" width="933" height="149" class="alignnone size-full wp-image-595" /></a>

<h5>Condition #2 - Invalid Job Name</h5>
<pre class="lang:ps decode:true " title="Bad JobName parameter" >
# Setup pathing and environment based on the script location
$Invocation = (Get-Variable MyInvocation -Scope 0).Value
$ScriptLocation = Split-Path $Invocation.MyCommand.Path
 
# Load the Start-SQLAgentJob cmdlet
. "$ScriptLocation\Start-SQLAgentJob.ps1"

# Set a couple variables for testing and call the cmdlet
$SQLServer = "localhost\inst1"
$JobName = "BadJobName"

$ErrorActionPreference = "Stop"
Start-SQLAgentJob -SQLServer $SQLServer -JobName $JobName
</pre> 
<a href="/img/2015/05/SQLAgent_BadJobName.png"><img src="/img/2015/05/SQLAgent_BadJobName.png" alt="SQLAgent_BadJobName" width="846" height="150" class="alignnone size-full wp-image-594" /></a>

<h5>Condition #3 - The job failed to run successfully</h5>
For this test case I explicitly set my SQL Agent Job to fail all the time in the job step.
<a href="/img/2015/05/SQLAgent_JobAlwaysFails.jpg"><img src="/img/2015/05/SQLAgent_JobAlwaysFails.jpg" alt="SQLAgent_JobAlwaysFails" width="506" height="156" class="alignnone size-full wp-image-587" /></a>

<pre class="lang:ps decode:true " title="Always Failing Job" >
# Setup pathing and environment based on the script location
$Invocation = (Get-Variable MyInvocation -Scope 0).Value
$ScriptLocation = Split-Path $Invocation.MyCommand.Path
 
# Load the Start-SQLAgentJob cmdlet
. "$ScriptLocation\Start-SQLAgentJob.ps1"

# Set a couple variables for testing and call the cmdlet
$SQLServer = "localhost\inst1"
$JobName = "TestJob"

$ErrorActionPreference = "Stop"
Start-SQLAgentJob -SQLServer $SQLServer -JobName $JobName
</pre>
<a href="/img/2015/05/SQLAgent_FailedJob.png"><img src="/img/2015/05/SQLAgent_FailedJob.png" alt="SQLAgent_FailedJob" width="915" height="149" class="alignnone size-full wp-image-596" /></a>

That's the end of the testing I need to do at this point. I have demonstrated that the cmdlet throws the appropriate error for each of the error conditions that I want to catch.

<h5>Conclusion</h5>
I can't stress enough how important error generation and error handling are when it comes to automation. Errors are the only mechanism that your scripts have to communicate that something went wrong back to the calling program or script. If scripts and cmdlets don't throw errors when they should, we have to learn to build that functionality into them. And never assume that errors are going to be thrown when you need them. Some errors in PowerShell are non-terminating and sometimes they just don't get thrown like you think they should. That is why it is very important to test each and every error condition to make sure your scripts behave like you want them to. 

This is my solution for this problem, but it's definitely not the only one. Thankfully, PowerShell gives you the flexibility to tailor your solution to your needs. Look back next week for my 3rd post in this series - SQL Agent Job Wrapper Part 3 - Handing the Errors in the Wrapper Script. 

Happy scripting!
