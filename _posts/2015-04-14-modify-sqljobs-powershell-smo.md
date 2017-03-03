---
layout: post
title: Modify SQL Agent Jobs using PowerShell and SMO
date: 2015-04-14 08:15
author: cjsommer@gmail.com
comments: true
categories: [PowerShell, SQL Server]
---
So here we are, week 2 of the <a href="http://www.edleightondick.com/2015/03/sql-new-blogger-challenge/" title="#SQLNewBlogger Challenge" target="_blank">#SQLNewBlogger Challenge</a>. This is a follow on to last weeks post <a title="Monitoring SQL Agent Jobs when you work for Mr Krabs" href="http://www.cjsommer.com/mrkrabs-sqlagent-job-monitoring/" target="_blank">Monitoring SQL Agent Jobs when you work for Mr Krabs</a> where I showed you how to go about monitoring SQL Server agent jobs using PowerShell and SMO. This can be very helpful if you are on a limited budget and can't afford any fancy monitoring tools. 

This week I have decided to stick to the same subject, but I'm gonna kick it up a notch. This week I am going to show you to how to modify SQL Server agent jobs with PowerShell and SMO. 

There are a number of operations we routinely perform when we are working with SQL Agent jobs. We add steps to them, setup schedules, configure output file locations and setup notifications. Most DBA's are quite familiar with using SQL Server Management Studio for performing these tasks. I am here to show you that you can do the same thing with PowerShell and SMO, and in some cases it can save you a hell of a lot of time.

Why would I want to use a PowerShell script instead of just managing the jobs through SSMS? I'm sure there are more, but here are a few use cases I can think of.
<ol>
	<li>You are migrating your database from a standalone server to SQL 2012 instance where the database will be added to an Always On Availability Groups. If you want your jobs to run flawlessly against a database that is in an Availability Group you need to add a step to all of your jobs to ensure that they are only running against the primary replica. There are exceptions to this rule, but for most jobs in my environment this step is very much required.</li>
	<li>When you want to set/change the output location for all of the jobs on a SQL Server. Maybe you copied the jobs between servers and the output file location changed. Or maybe someone forgot to setup output files for some jobs. If you have a lot of jobs this can be quite the tedious process, and also prone to errors.</li>
	<li>Disabling or enabling jobs en mass or through an automated process.</li>
</ol>
All of these tasks can be very painful if you have to manually edit the jobs using SSMS, especially if you have a lot of jobs. I know this because I faced this exact task just last week. I had to add a job step to all of the SQL Agent jobs for the Availability Group check (as mentioned above), and I also had to add an output file location for each of the jobs. 

I performed this task manually with another DBA. We had 25 or so jobs to go through and I honestly did't think it would take as long as it did. After we got rolling we ended up making a bunch of copy/paste errors and we also realized that we weren't consistent in our naming conventions, all of which created a bunch of rework for us. Two hours later my fingers were bored, my brain was slowly turning to mush and my eyeballs hurt. It was at this point that I vowed, never again, and decided to come up with a more automated solution.

For the example scripts below I will be focusing specifically on setting the default output file location for each SQL Agent job step, but the abilities of SMO do not end there. I think it's worth reiterating this fact. Everything you can to SQL Agent jobs using SSMS, you can do using PowerShell and SMO. 

The foundation once again will be the SQLPS module, which gives us access to the SMO libraries as well as the SQL Provider which are both requirements of the example below. Let's dive in!

<hr />

The core object class in this demo is the microsoft.sqlserver.management.smo.agent.jobstep. This class gives us access to all of the methods and properties we need to manage SQL Server agent job steps. If you load SQLPS and create a JobStep object, you can look at the properties and methods by using the Get-Member cmdlet. 

Note: You can also visit <a title="MSDN JobStep Class" href="https://msdn.microsoft.com/en-us/library/microsoft.sqlserver.management.smo.agent.jobstep.aspx" target="_blank">MSDN JobStep Class</a> to get help online. I sometimes find the MSDN class pages a bit easier to navigate for the more complex objects.

<pre class="theme:powershell-ise toolbar:1 nums:false scroll:true tab-convert:true lang:ps decode:true" title="Load SQLPS and create a JobStep object">

Push-Location; Import-Module SQLPS -DisableNameChecking; Pop-Location;
$JobStep = New-Object microsoft.sqlserver.management.smo.agent.jobstep

</pre>

<hr />

The properties contain all of the possible configuration items that we can manipulate in each JobStep object. In this example I will be manipulating the OutputFileName as well as the JobStepFlags property, where I will be adding the configuration value "AppendToLogFile".
<pre class="theme:powershell-ise toolbar:1 nums:false scroll:true tab-convert:true lang:ps decode:true" title="Get the JobStep object properties"># Display all of the properties that come with the JobStep object
$JobStep | Get-Member -MemberType Property
</pre>
<a href="http://www.cjsommer.com/wp-content/uploads/2015/04/JobStepProperties.jpg"><img class="alignnone size-full wp-image-160" src="http://www.cjsommer.com/wp-content/uploads/2015/04/JobStepProperties.jpg" alt="JobStepProperties" width="965" height="519" /></a>

<hr />

The one method I am using in the script below is the Alter() method. After you set the properties to the values you want, you have to run the Alter() method to apply the changes. Without the Alter() they will go back to what they were previously. Also listed below are all of the other methods that come with the JobStep object.
<pre class="theme:powershell-ise toolbar:1 nums:false scroll:true tab-convert:true lang:ps decode:true" title="Get the JobStep object methods">$JobStep | Get-Member -MemberType Method
</pre>
<a href="http://www.cjsommer.com/wp-content/uploads/2015/04/JobStepMethods.jpg"><img class="alignnone size-full wp-image-159" src="http://www.cjsommer.com/wp-content/uploads/2015/04/JobStepMethods.jpg" alt="JobStepMethods" width="964" height="458" /></a>

<hr />

Below is a complete working script. To use it you need to set the $SQLServer and $SQLAgentOutputLocation variables for the server you will be running it against. $SQLServer is the name of the SQL Server and $SQLAgentOutputLocation is the output file location where all the job step output will end up. I highly recommend running it against a DEV instance and understanding it before running against PROD because as it sits, it WILL change all of your job step locations. There is a way to disable that if you are so inclined. Hint: Just comment out the Alter() line. ;)

Another thing to note is that the job step output filenames are setup to use tokens. Based on this script, the OutputFileNames will get set to "JobName_Step1_yyyyMMdd.txt". The JobName comes from SMO, but the  step number and date are dynamic because they use tokens. Check out the MSDN article for <a href="https://msdn.microsoft.com/en-us/library/ms175575.aspx" title="Using Tokens in Job Steps" target="_blank">Using Tokens in Job Steps</a> for more details. You can most certainly customize the filenames to whatever standard you desire. Enough chatter, here's the script!

<pre class="theme:powershell-ise toolbar:1 nums:false scroll:true tab-convert:true lang:ps decode:true" title="Full Script">Push-Location; Import-Module SQLPS -DisableNameChecking; Pop-Location;

# Set SQLServer a SQLAgentOutputLocation to fit your needs
# If it is the default instance us SERVERNAME\DEFAULT as the SQLServer
$SQLServer = "MYSERVER\MYINSTANCE"
$SQLAgentOutputLocation = "C:\SQLJobOutput"

$SQLProviderLocation = "SQLSERVER:\SQL\${SQLServer}\JobServer\Jobs"

# Get all of the job steps and hold the output in the $JobSteps object
$JobSteps = New-Object microsoft.sqlserver.management.smo.agent.jobstep
$JobSteps = Get-ChildItem -Path $SQLProviderLocation | %{$_.enumjobstepsbyid()}

# Go through each job step and change the OutputFileName to the new location, and also set it to Append Output
foreach ($Step in $JobSteps) {
    $Parent = $Step.parent
    $StepName = $Step.Name
    $StepOutputFIle = $step.OutputFileName
    $JobName = ($Parent.Name).Replace(" ","")
    $StepOutFile = "${SQLAgentOutputLocation}\${JobName}" + '_Step$(ESCAPE_SQUOTE(STEPID))_$(ESCAPE_SQUOTE(STRTDT)).txt'
    $Step.OutputFileName = $StepOutFile
    $Step.JobStepFlags = "AppendToLogFile"

    # The Alter() call actually modifies the value
    $Step.Alter()

    # Display what the new job step details are
    $Parent.Name
    $Step.Name
    $Step.OutputFileName
    $Step.JobStepFlags
    " "
}
</pre>

Looking back at the 2 hours I lost last week I can safely say that there is no reason to manually modify 25 jobs just to change the output file location. Honestly the time it took me to come up with this script was less than the time it took me to manually modify those jobs, and PowerShell did it correctly the first time. The best part of writing snippets like this is that I never have to do it again. I now have something I can use in the future.

Just like the SQL Agent monitor script in my last blog post, this is a good script to run against a single server. You are welcome to bind, spindle or mutilate it script to fit your needs. My hope in sharing is that you might see the potential of using PowerShell and SMO to help automate this type of task.

Happy scripting!
