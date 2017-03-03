---
layout: post
title: T-SQL Tuesday #68: Just Say No to Defaults
date: 2015-07-14 08:00
author: cjsommer@gmail.com
comments: true
categories: [PowerShell, SQL Server]
---
<h1>Using PowerShell and SMO to set MinServerMemory and MaxServerMemory Programatically</h1>

<a href="https://sqlbek.wordpress.com/2015/07/06/invitation-to-t-sql-tuesday-68-just-say-no-to-defaults/" target="_blank"><img src="http://www.cjsommer.com/wp-content/uploads/2015/05/TSQLTuesday.jpg" alt="TSQLTuesday" width="150" height="150" class="alignright size-full wp-image-504" /></a>
Here is a link to the official #tsql2sday invitation from <a href="https://sqlbek.wordpress.com/2015/07/06/invitation-to-t-sql-tuesday-68-just-say-no-to-defaults/" target="_blank">Andy Yun's Blog</a>. This month's subject is "Just Say No to Defaults". If you've read my blog at all you will know that I have a slight obsession with SQL Server and PowerShell. As far as I am concerned PowerShell is the new gold standard when it comes to scripting and automation in the Windows environment. Add a sprinkle of SQL Server with the SQL Server PowerShell module  (SQLPS) and you have a deadly combination as a DBA. 

So lets start saying no to defaults! One setting I always change when I install SQL Server are the MinServerMemory and MaxServerMemory values. This blog post will demonstrate how to use PowerShell and SMO to set these values programmatically, based on the amount of RAM in the machine you are running it on. 

<hr>
<h3>Load SQLPS and Create a Server Object</h3>
This first code block will load SQLPS module, connect to the SQL Server instance of your choosing, and test the connection before moving on. The SQLPS module gives us access to the SQL Management Object libraries (a.k.a SMO), which is our interface into SQL Server. Any configuration property that you can change using SQL Management Studio (SSMS), you can also change using SMO.
<pre class="lang:ps decode:true " title="Load SQLPS and Create a Server Object" >
# Load SQLPS
Push-Location; Import-Module SQLPS -DisableNameChecking; Pop-Location

# Microsoft.SqlServer.Management.Smo.Server
$SqlServerName = "localhost\inst1"
$SrvObject = New-Object 'Microsoft.SqlServer.Management.Smo.Server' $SQLServerName

# Test the connection
$SrvObject.ConnectionContext.Connect()
</pre>

<hr>
<h3>Find the Memory Configuration Object Names</h3>
Next we need to find the correct configuration properties to work within SMO. I am not going to go into all of the properties and members of the SMO server object as there are a lot of them. I know from experience that MinServerMemory and MaxServerMemory are ConfigProperty objects under Configuration property. Wow that sounds really confusing. Hopefully the following code snippet will shed some light on it.
<pre class="lang:ps decode:true " title="Find the Memory Configuration Object Names" >
# Find the correct configuration property for memory
$SrvObject.Configuration | Get-Member *memory*
</pre> 
<img src="http://www.cjsommer.com/wp-content/uploads/2015/07/tsql2sday68_getmemprop.jpg" alt="tsql2sday68_getmemprop" width="773" height="206" class="alignnone size-full wp-image-787" />

<hr>
<h3>Display the current MinServerMemory and MaxServerMemory Values</h3>
MinServerMemory and MaxServerMemory are the properties we are looking for. If you run the following snippet you will be able to see the existing values for the MinServerMemory and MaxServerMemory for your SQL Server.
<pre class="lang:ps decode:true " title="Display the current MinServerMemory and MaxServerMemory Values" >
# Display the Min and MaxServerMemory configuration properties
$SrvObject.Configuration.MinServerMemory 
$SrvObject.Configuration.MaxServerMemory 
</pre> 

<img src="http://www.cjsommer.com/wp-content/uploads/2015/07/tsql2sday68_maxservermemory.png" alt="tsql2sday68_maxservermemory" width="393" height="138" class="alignnone size-full wp-image-790" />

Only MaxServerMemory is shown in the output, but the MinServerMemory properties are identical. Actually, the properties for all of the SQL Server Configuration items are the same. They all contain the following 9 properties.
<ul>
	<li>DisplayName - Friendly display name</li>
	<li>Number - A unique number that identifies this configuration setting.</li>
	<li>Minimum - The minimum value</li>
	<li>Maximum - The maximum value</li>
	<li>IsDynamic - 'True' - changes take effect immediately; 'False' - changes need a recycle of SQL Server.</li>
	<li>IsAdvanced - Is this an advanced configuration setting?</li>
	<li>Description - Friendly description of the configuration</li>
	<li>RunValue - Current running value</li>
	<li>ConfigValue - Current configured value</li>
</ul>

<hr>
<h3>Set MinServerMemory and MaxServerMemory</h3>
Setting the configuration is fairly straight forward. As seen in the next code snippet, all you need to do is set the config value and then run the alter method on the configuration property. That's it. These 2 commands don't output anything so to check the result you will have to run the display snippet to verify that the values have been changed.
 
<pre class="lang:ps decode:true " title="Set MinServerMemory and MaxServerMemory"  >
# Set the MaxServerMemory
$SrvObject.Configuration.MaxServerMemory.ConfigValue = 128
$SrvObject.Configuration.MaxServerMemory.ConfigValue = 512
$SrvObject.Configuration.Alter()</pre> 

<hr>
<h3>Dynamically Set Min and Max Server Memory - Full Script</h3>
OK, that's cool and all, but lets put some dynamic logic in the script. Lets say I want to set my MaxServerMemory at 90% of the total physical memory in the server, and I want to set the MinServerMemory at 50% of the MaxServerMemory. Because we're using PowerShell, we can use WMI to get the physical server memory and then perform some simple calculations to set the SQL Server memory settings more dynamically. Below is the full script that you can use just for this purpose.
 
<pre class="lang:ps decode:true " title="Dynamically Set Min and Max Server Memory - Full Script" >
# Programmatically Alter the MinServerMemory and MaxServerMemory

# Load SQLPS
Push-Location; Import-Module SQLPS -DisableNameChecking; Pop-Location
 
# Microsoft.SqlServer.Management.Smo.Server
$SqlServerName = "localhost\inst1"
$SrvObject = New-Object 'Microsoft.SqlServer.Management.Smo.Server' $SQLServerName
 
# Test the connection
$SrvObject.ConnectionContext.Connect()

# Get total physical memory in the machine. There's more than 1 way to skin this cat.
$TotalRam = [int]((Get-WmiObject Win32_OperatingSystem | Select-Object -ExpandProperty TotalVisibleMemorySize) / 1024)

# Set the Max to 90% of physical memory and Min to 1/2 of Max memory.
[int]$MaxMem = $TotalRam * .9
[int]$MinMem = $MaxMem / 2

# Alter the SQL Server configuration values
$SrvObject.Configuration.MaxServerMemory.ConfigValue = $MaxMem
$SrvObject.Configuration.MinServerMemory.ConfigValue = $MinMem
$SrvObject.Configuration.Alter()

# Display the new values
$SrvObject.Configuration.MaxServerMemory
$SrvObject.Configuration.MinServerMemory
</pre> 

The full script displays the MinServerMemory and MaxServerMemory configuration values after the modification so you can see that your script worked as expected.

<img src="http://www.cjsommer.com/wp-content/uploads/2015/07/tsql2sday68_dynamic.png" alt="tsql2sday68_dynamic" width="399" height="277" class="alignnone size-full wp-image-818" />

And there you have it. You can use PowerShell and SMO to set your MinServerMemory and MaxServerMemory settings based on the physical amount of RAM in your system. As a disclaimer, my calculations for MinServerMemory and MaxServerMemory is just what I used for this demo and in no way a standard for everything. I just needed a simple example to show. You could make the logic as simple or complex as you like. That part is up to you to figure out!

Just as with SQL Server itself, you can also manage SQL Server Agent configurations using PowerShell and SMO. Here is a link to that blog post for <a href="http://www.cjsommer.com/sql-agent-smo/" target="_blank">Managing SQL Server Agent Configuration with PowerShell</a> in case you're interested. 

Enjoy and happy scripting!
