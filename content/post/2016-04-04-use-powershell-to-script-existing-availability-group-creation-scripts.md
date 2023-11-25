---
layout: post
title: Use PowerShell to script existing Availability Group creation scripts!
date: 2016-04-04T10:00:00
author: cjsommer@gmail.com
comments: true
tags: ["Powershell", "SQL Server"]
---
Scripts that write scripts! One of my favorites!

So last night I was up working on an issue with an Availability Group. For whatever reason the cluster resource for the AG would not come online. Looking in SSMS the AG's on both replicas were stuck in "Resolving". I beat on the keyboard for almost 2 hours before deciding that rebuilding the AG was my best option. The databases were down and the birds were starting to chirp and I needed to get them back online.

If you've ever built an Availability Group before you know it's not horribly difficult, but you need some critical pieces of information. Replica Names, Endpoint Port Numbers, Availability Group Name, Listener Name, Listener IP's and all of the configurations that go along with it.

So I found myself scrounging for this information in the wee hours of the morning, hoping I wouldn't miss anything before I removed the AG from SQL Server. In the words of my old boss, "hope is not a strategy". Thankfully I had captured all of the information I needed and was able to rebuild the AG successfully. Problem solved...this time.

In retrospective I wish I would have had scripts to recreate the AG's. I know not everyone likes to script out all of their work. Some people prefer the GUI, that's fine. But at 2:00 AM the last thing I want to be searching for is this information in hopes that I get it all correct. I know someone is going to say "it should be in the run book or in the build documentation". If that documentation is static there's a good chance that it is out of date. Static documentation is only good as long as people are religious about maintaining it, and on a team of 8 DBA's that's usually not the case. I like dynamic documentation. Documentation that is created from the live, running system. 

Which brings me to the point of this blog post. I decided after last night's "fun" that I would create a script to create a script for rebuilding my AG's. As I said at the beginning, scripts that create scripts are one of my favorite things to do.

So I started down the T-SQL path and in case you haven't worked with the HADR system tables yet, let me just say it takes a while to find the info you need sometimes. The information you need is in there, but it usually takes me multiple JOINS and multiple cuss words before I get a query that does what I need it to do. Being a PowerShell guy a light bulb went off. I have used the SMO scripter object to script database objects before. What if I could script out AG objects as well? Sure enough, the answer is yes!
<hr>
Here is the PowerShell code that I came up with. It goes through and scripts out the definitions of every AG on a server to a T-SQL script. It works with single or multiple Availability Groups. Here is how you use it.
<ol>
	<li>Copy ScriptMyAvailabilityGroups.ps1 and save it to your system somewhere.</li>
	<li>Set the $SQLServer variable to one of your SQL Servers that has AG's on it.</li>
	<li>Run it!</li>
</ol>
 
```powershell
# SQL Server you want to run this against
$SQLServer = 'SQLINST1\INST1'

# Setup pathing and environment based on the script location
$Invocation = (Get-Variable MyInvocation -Scope 0).Value
$ScriptLocation = Split-Path $Invocation.MyCommand.Path
$ScriptName = $Invocation.MyCommand.Name.Replace(".ps1","")
$ScriptFullPath = $Invocation.MyCommand.Path

# Load SMO
[System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SqlServer.SMO") | out-null

$SQLObj = New-Object "Microsoft.SqlServer.Management.Smo.Server" $SQLServer
$SQLObj.ConnectionContext.Connect()

foreach ($ag in ($SQLObj.AvailabilityGroups )){
    $SQLINST = $SQLServer.Replace('\','_')
    $AGName = $ag.Name
    $Dttm = (Get-Date -Format 'yyyyMMdd_hhmm')

    $OutFile = "$ScriptLocation\AGInfo\$SQLINST\${AGname}_${Dttm}.sql"
    if (!(Test-Path -Path $OutFile -PathType Leaf)) {
        New-Item -Path $OutFile -ItemType File -Force
    }
    Write-output "Scripting Availability Group [$AGName] to '$OutFile'"

    '/*' | Out-File -FilePath $OutFile -Encoding ASCII -Force
    $ag | Select-Object -Property * | Out-File -FilePath $OutFile -Encoding ASCII -Append
    '*/' | Out-File -FilePath $OutFile -Encoding ASCII -Append

    $scriptr = new-object ('Microsoft.SqlServer.Management.Smo.Scripter') ($SQLObj)
    $scriptr.Script($ag) | Out-File -FilePath $OutFile -Encoding ASCII -Append
}
```

The PowerShell script saves the T-SQL script in a sub-directory under the location where ScriptMyAvailabilityGroups.ps1 lives.

 
```powershell
C:\Users\cjsommer\Documents\WindowsPowerShell\AGInfo\SQLINST1_INST1 [master +3 ~1 -0 !]> ls

    Directory: C:\Users\cjsommer\Documents\WindowsPowerShell\AGInfo\SQLINST1_INST1

Mode                LastWriteTime     Length Name
----                -------------     ------ ----
-a---        2016-04-01     13:41       2388 MYAG1_20160401_0139.sql

C:\Users\cjsommer\Documents\WindowsPowerShell\AGInfo\SQLINST1_INST1 [master +3 ~1 -0 !]>
```

And here are the contents of the T-SQL script that was generated.
 
```sql
Parent                     : [SQLINST1\INST1]
AutomatedBackupPreference  : Secondary
BasicAvailabilityGroup     : 
DatabaseHealthTrigger      : 
DtcSupportEnabled          : 
FailureConditionLevel      : OnCriticalServerErrors
HealthCheckTimeout         : 30000
ID                         : 65536
LocalReplicaRole           : Primary
PrimaryReplicaServerName   : SQLINST1\INST1
UniqueId                   : 646cf943-a4b5-406d-8c75-29ffd7e6424e
AvailabilityReplicas       : {SQLINST1\INST1, SQLINST2\INST1}
AvailabilityDatabases      : {UserDB1, UserDB2...}
DatabaseReplicaStates      : {MYAG1, MYAG1, MYAG1, MYAG1...}
AvailabilityGroupListeners : {MYLSNR1}
Name                       : MYAG1
Urn                        : Server[@Name='SQLINST1\INST1']/AvailabilityGroup[@Name='MYAG1']
Properties                 : {Name=AutomatedBackupPreference/Type=Microsoft.SqlServer.Management.Smo.AvailabilityGroupAutomatedBackupPreference/Writable=True/Value=Secondary, 
                             Name=FailureConditionLevel/Type=Microsoft.SqlServer.Management.Smo.AvailabilityGroupFailureConditionLevel/Writable=True/Value=OnCriticalServerErrors, 
                             Name=HealthCheckTimeout/Type=System.Int32/Writable=True/Value=30000, Name=ID/Type=System.Int32/Writable=False/Value=65536...}
ExecutionManager           : Microsoft.SqlServer.Management.Smo.ExecutionManager
UserData                   : 
State                      : Existing



*/
CREATE AVAILABILITY
GROUP [MYAG1]
WITH (
		AUTOMATED_BACKUP_PREFERENCE = SECONDARY
		,FAILURE_CONDITION_LEVEL = 3
		,HEALTH_CHECK_TIMEOUT = 30000
		)
FOR DATABASE [UserDB1]
	,[UserDB2] REPLICA ON N'SQLINST1\INST1'
WITH (
		ENDPOINT_URL = N'TCP://SQLINST1.mydomain.com:5022'
		,FAILOVER_MODE = MANUAL
		,AVAILABILITY_MODE = SYNCHRONOUS_COMMIT
		,SESSION_TIMEOUT = 10
		,BACKUP_PRIORITY = 50
		,PRIMARY_ROLE(ALLOW_CONNECTIONS = ALL)
		,SECONDARY_ROLE(ALLOW_CONNECTIONS = NO)
		)
	,N'SQLINST2\INST1'
WITH (
		ENDPOINT_URL = N'TCP://SQLINST2.mydomain.com:5022'
		,FAILOVER_MODE = MANUAL
		,AVAILABILITY_MODE = SYNCHRONOUS_COMMIT
		,SESSION_TIMEOUT = 10
		,BACKUP_PRIORITY = 50
		,PRIMARY_ROLE(ALLOW_CONNECTIONS = ALL)
		,SECONDARY_ROLE(ALLOW_CONNECTIONS = NO)
		) LISTENER N'MYLSNR1' (
		WITH IP((
					N'192.168.0.100'
					,N'255.255.255.0'
					))
			,PORT = 1433
		);
```

So that's that. It's a pretty simple PowerShell script that provide you with an up to date T-SQL script that will be useful if you ever end up having to rebuild your AGs at 2:00 AM like I did last night. There is obviously (or not so obviously) a lot of things you could do to customize it and make it your own. Enjoy and happy scripting!
