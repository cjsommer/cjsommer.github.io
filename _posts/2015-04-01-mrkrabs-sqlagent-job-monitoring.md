---
layout: post
title: Monitoring SQL Agent Jobs when you work for Mr Krabs
date: 2015-04-01 21:13
author: cjsommer@gmail.com
comments: true
categories: [PowerShell, SQL Server]
---
For those of you who don't know who Mr. Krabs is, he is a character in the TV show Sponge Bob Square Pants. Mr. Krabs owns the Krusty Krab restaurant and is a very frugal business owner. Every decision he makes is driven completely on how it will impact his bottom line.

Thinking about Mr. Krabs reminds me of one of my first SQL Server DBA jobs. I was starting just as the old DBA was leaving and there wasn't a lot of time for turnover. I was the only DBA on staff, my servers were old, the CPU's slow, the disk drives were small, and none of it was being monitored! Yikes! Monitoring was one thing I knew I had to address right away, but I didn't have a budget to work with! The business just wasn't willing to spend the money on a fancy monitoring application. They were my Mr. Krabs!

As DBA's we are responsible for making sure our SQL Servers are operationally sound. We need to ensure that all of the day-to-day maintenance tasks are running (backups, index rebuilds, update stats). We have to ensure the same for any application specific jobs (data feeds, purges, cleanup, etc.). When you don't have any money to buy fancy monitoring tools you have to get creative, and that's just what I did. I rolled up my sleeves and wrote my own SQL Agent Job monitoring solution. Innovation and creativity were a way of life and I learned a TON along the way!

If you're strapped on cash, I am here to show you that you don't need expensive monitoring tools keep watch over your SQL Agent jobs. For me, PowerShell coupled with the SQL Server Module (SQLPS) were all I needed. If you don't have other monitoring tools in place this just might be what you are looking for.
<h3>What do you need to get started?</h3>
<ol>
	<li>PowerShell 4.0. If you're on 2.0 or 3.0, upgrade. It's worth it!</li>
	<li>SQL Server Module SQLPS. Installed when you install SQL Server or SSMS.</li>
	<li>A SQL Server. Preferably with some jobs running on it.</li>
</ol>

<hr />

If you're not familiar with SMO, they are the SQL Management Objects. They let you manage SQL Server. Instance configuration, database settings and a whole bunch of other really cool stuff including SQL Agent Jobs.

The easiest way to get access to the SMO objects is just to load SQLPS module. There are other ways to load individual assemblies, but for the purpose of this post, lets just load the SQLPS module.
<pre class="theme:powershell-ise toolbar:1 nums:false scroll:true tab-convert:true lang:ps decode:true" title="Load the SQLPS Module">Push-Location; Import-Module SQLPS -DisableNameChecking; Pop-Location ;
</pre>

<hr />

Next, we are going to set some variables for use in the script. The SQLServer is the name of the SQL Server we are going to interrogate. The BeginDate and EndDate are the date ranges for the job history and are used to limit the result set. This helps greatly with performance of the script if you keep a lot of SQL Job History.
<pre class="theme:powershell-ise toolbar:1 nums:false scroll:true tab-convert:true lang:ps decode:true" title="Set your variables"># Set our Variables
[string]$SQLServer = "localhost\inst1"
[datetime]$BeginDate = "2015-04-01 00:00:00"
[datetime]$EndDate = Get-Date
</pre>

<hr />

Next is to create the SMO Server Object. This is our connection to SQL Server using SMO. The ConnectionContext.Connect() statement tests the connection and is our validation that we have connected to an active SQL Server.
<pre class="theme:powershell-ise toolbar:1 nums:false scroll:true tab-convert:true lang:ps decode:true" title="Create an SMO server object"># Create an SMO Server object and initialize some variables for use in the loop
$oServer = New-Object "Microsoft.SqlServer.Management.Smo.Server" $SQLServer ;
$oServer.ConnectionContext.Connect() ;
</pre>

<hr />

This next block of code does all of the work for us. It gets the history of each job on the server that falls in between the BeginDate and the EndDate. All of the history that meets that criteria is then added to the variable $JobHistory. That variable will contain all of the JobHistory objects once this portion of the script is done.
<pre class="theme:powershell-ise toolbar:1 nums:false scroll:true tab-convert:true lang:ps decode:true" title="Get the job history"># Get the job history for each job
$job = $null ;
$JobHistory = $null ;
foreach($job in $oServer.JobServer.Jobs | 
    Where-Object { ($_.IsEnabled -eq $true) `
                -and ($_.LastRunDate -ge $BeginDate) `
                -and ($_.LastRunDate -le $EndDate) } ) 
{          
    $tmpJobHistory = $null ;
    $tmpJobHistory = $job.EnumHistory() | 
        Where-Object { ($_.StepID -eq "0") `
                    -and ($_.RunDate -ge $BeginDate) `
                    -and ($_.RunDate -le $EndDate) } ;
    $JobHistory += $tmpJobHistory ;
}
</pre>

<hr />

This last little block of code outputs the results to us. The SQL Agent RunStatus is a simple integer value. The snippet below changes that integer to a more readable format using a hash table. The JobHistory variable is an array of JobHistory objects and contains all of the properties that come with it. The final output statement selects only a few of the columns from the JobHistory variable and displays it using the Format-Table cmdlet. This is a very typical way of displaying your output.
<pre class="theme:powershell-ise toolbar:1 nums:false scroll:true tab-convert:true lang:ps decode:true" title="Display our Job History"># Fancy way to remap the RunStatus, which is an integer, to human readable text
$RunStatus = @{'0'="Failed";'1'="Succeeded";'2'="Retry";'3'="Cancelled"}
$RunStatusColumn = @{
    Name = 'RunStatus'
    Expression = { $RunStatus.($_.RunStatus.ToString()) }
}

# Now Output the Results. Notice the reference to the hash table for RunStatus
$JobHistory | Select-Object JobName,RunDate,$RunStatusColumn | 
    Sort-Object JobName,RunDate | Format-Table -AutoSize ;
</pre>

<hr />

So if I put all of those pieces together and I run it, here is the output that is generated for the instance I am running it against right now.

<a href="http://www.cjsommer.com/wp-content/uploads/2015/04/PrtScr.png"><img class="alignnone size-full wp-image-119" src="http://www.cjsommer.com/wp-content/uploads/2015/04/PrtScr.png" alt="PowerShell Output" width="661" height="161" /></a>

This is just a snippet of the output, but I am hoping you get the idea. If you run the script against one of your SQL Servers you should see all of the job history between the start and end dates you set. You can run it against a single server manually, or it can be used as the foundation for a more automated solution. It's really just the tip of the iceberg and ready to be molded to the way you want it to.

Initially, I was kind of forced down the path of creating my own monitoring solutions due to budgetary constraints. I am at a new company now and we do have a lot of really fancy tools in place, but I still turn to PowerShell and SMO for a lot of my every day tasks. It's a very potent combination!

The full script is attached to this post below. It's attached as txt file because Wordpress complained. Just rename it to ps1, set the variables to fit your needs and you should be good to go.

Happy scripting!
