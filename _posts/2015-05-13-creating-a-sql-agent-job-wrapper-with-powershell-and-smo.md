---
layout: post
title: Creating a SQL Agent Job Wrapper with PowerShell and SMO
date: 2015-05-13 08:00
author: cjsommer@gmail.com
comments: true
categories: [PowerShell, SQL Server]
---
<h2>Some Background</h2>
"Is there any way to start a SQL Agent job using a script or program?"

I've seen this question asked a number of times over the past couple weeks and I thought I would share an approach using PowerShell and SMO. But first, let's take a step back and try to understand why someone would want to do this. What's wrong with SQL Agent? 

Well there's really nothing wrong with SQL Agent. The main reason I have seen people asking this question is because their company is looking into using an enterprise job scheduler. An enterprise job scheduler gives an operations group a single location to manage jobs across their whole environment. It allows them to see all the moving parts from a batch processing perspective, even across dissimilar platforms. It also allows them to create more complex workflows across multiple platforms. 

So if a company is looking to switch to an enterprise job scheduling solution they will need a way to transition any of their existing SQL Agent jobs to run through the new scheduler. One way to accomplish this is to create a wrapper. A wrapper is nothing more than a script or program that calls another script or program, or in this case, a SQL Agent job.

<h2>Test SQL Agent Job</h2>
The first thing I needed was a job to test with, so I created the following job on my local SQL instance. All it does is selects @@SERVERNAME, sleeps for 10 seconds and then exits, so it's pretty simple. Here is the TSQL to create that job.

```sql
USE [msdb]
GO

/****** Object:  Job [TestJob]    Script Date: 5/10/2015 9:06:29 PM ******/
BEGIN TRANSACTION
DECLARE @ReturnCode INT
SELECT @ReturnCode = 0
/****** Object:  JobCategory [[Uncategorized (Local)]]    Script Date: 5/10/2015 9:06:29 PM ******/
IF NOT EXISTS (SELECT name FROM msdb.dbo.syscategories WHERE name=N'[Uncategorized (Local)]' AND category_class=1)
BEGIN
EXEC @ReturnCode = msdb.dbo.sp_add_category @class=N'JOB', @type=N'LOCAL', @name=N'[Uncategorized (Local)]'
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback

END

DECLARE @jobId BINARY(16)
EXEC @ReturnCode =  msdb.dbo.sp_add_job @job_name=N'TestJob', 
		@enabled=1, 
		@notify_level_eventlog=0, 
		@notify_level_email=0, 
		@notify_level_netsend=0, 
		@notify_level_page=0, 
		@delete_level=0, 
		@description=N'No description available.', 
		@category_name=N'[Uncategorized (Local)]', 
		@owner_login_name=N'sa', @job_id = @jobId OUTPUT
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
/****** Object:  Step [TestStep]    Script Date: 5/10/2015 9:06:30 PM ******/
EXEC @ReturnCode = msdb.dbo.sp_add_jobstep @job_id=@jobId, @step_name=N'TestStep', 
		@step_id=1, 
		@cmdexec_success_code=0, 
		@on_success_action=1, 
		@on_success_step_id=0, 
		@on_fail_action=2, 
		@on_fail_step_id=0, 
		@retry_attempts=0, 
		@retry_interval=0, 
		@os_run_priority=0, @subsystem=N'TSQL', 
		@command=N'select @@servername
WAITFOR DELAY ''00:00:10''', 
		@database_name=N'master', 
		@flags=0
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
EXEC @ReturnCode = msdb.dbo.sp_update_job @job_id = @jobId, @start_step_id = 1
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
EXEC @ReturnCode = msdb.dbo.sp_add_jobserver @job_id = @jobId, @server_name = N'(local)'
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
COMMIT TRANSACTION
GOTO EndSave
QuitWithRollback:
    IF (@@TRANCOUNT > 0) ROLLBACK TRANSACTION
EndSave:
GO
```

<h2>The SQL Agent Job Wrapper Cmdlet</h2>
The next piece I needed to create was a PowerShell cmdlet to do the work for me. The cmdlet is the meat of the this whole process. It will start a SQL Server Agent job and wait for completion before returning to the calling script. The cmldlet requires the SQL Server and the Job Name that you want to execute as parameters. It will only attempt to run the job if it is enabled and currently idle. If for some reason the job is already running it will still wait for completion, but it wont try to start it again. When the job completes the cmdlet will return the Name, CurrentRunStatus, LastRunOutcome, LastRunDate and the LastRunDurationSeconds (which could be modified to fit your needs).

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

    $ServerObj = New-Object Microsoft.SqlServer.Management.Smo.Server($SQLServer)
    $ServerObj.ConnectionContext.Connect()
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

    $JobObj | select Name,CurrentRunStatus,LastRunOutcome,LastRunDate,@{Name="LastRunDurationSeconds";Expression={$RunDuration}}
}
```

<h2>Example of Running the Cmdlet</h2>
And finally I needed to create a script to test the cmdlet. The test script loads the cmdlet by dot sourcing the Start-SQLAgentJob.ps1 file and then calls the cmdlet by passing it the SQL Server and the JobName as in the example below. It's pretty simple.

```powershell
# Setup pathing and environment based on the script location
$Invocation = (Get-Variable MyInvocation -Scope 0).Value
$ScriptLocation = Split-Path $Invocation.MyCommand.Path

# Load the Start-SQLAgentJob cmdlet
. "$ScriptLocation\Start-SQLAgentJob.ps1"

# Set a couple variables for testing and call the cmdlet
$SQLServer = "localhost\inst1"
$JobName = "TestJob"
Start-SQLAgentJob -SQLServer $SQLServer -JobName $JobName
```

And here are my results:<a href="/img/2015/05/SqlAgentWrapperOut.jpg"><img src="/img/2015/05/SqlAgentWrapperOut.jpg" alt="SqlAgentWrapperOut" width="516" height="178" class="alignnone size-full wp-image-519" /></a>

<h2>Test it out for yourself</h2>
Save the scripts above into the same directory. The test script is set to look for the cmdlet script in the same directory. You can test it against an existing job on your SQL Server or use the TestJob.sql to create the same one I used. Either way will work fine. The only script you should have to modify is the Test-Wrapper.ps1 script. Just set the SQLServer and JobName variables to one of your own SQL Server Jobs and let 'er rip!

<h2>So we're good, right?</h2>
I'd love to say that this is a complete solution, but it is not. It is missing a critical piece before I would say it is production ready, and that critical piece is error handling. It's great that we are able to kick off the SQL Agent Job from an external script and wait for it to return, but we also need to determine if it was successful or not. That will be the focus of my blog post for next week. Adding error handling to the SQL Agent Job Wrapper, so stay tuned!

As always, Happy Scripting!
