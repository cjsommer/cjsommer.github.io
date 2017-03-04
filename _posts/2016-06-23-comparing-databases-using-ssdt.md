---
layout: post
title: Comparing databases using SSDT
date: 2016-06-23 13:05
author: cjsommer@gmail.com
comments: true
categories: [PowerShell, SQL Server]
---
Another Twitter-born blog post! I love getting new ideas from the community. Real world ideas for solving real world problems!

<a href="https://twitter.com/Kevin3NF/status/746002230969536512" target="_blank"><img alt='' class='alignright size-full wp-image-1295 ' src='/img/2016/06/img_576c0c44b47c0.png' /></a>
The original question was "<em>Does SQL Compare or others prods allow me to compare 1 "gold" db to the 57 "identical" databases in prod at once? asking for a friend ;)</em>". The original question and link to the twitter feed is off to the right. 

The ask here was pretty straight forward. SQLCyclist needed to compare 57+ databases to one database which he considered his gold copy. Basically, how far have the schemas drifted from one another? I happen to use SSDT for doing something similar and I knew it might fit the bill for him. The script I do have includes some custom stuff for my applications that wouldn't be relevant, so I set out on a quest to see what I could come up with. 

If you haven't used SSDT you're missing out. It's a great tool provided by Microsoft for database development and deployment. As a matter of fact it's the only way to get to the cloud if you happen to venture into the stratosphere, but I digress. And away we go...

So on my local SQL Server I made 2 identical copies of my AdventureWorks2012 database (rest in peace AW).

<img alt='' class='alignnone size-full wp-image-1305 ' src='/img/2016/06/img_576c10b7a08b7.png' />

And then I got scripting. Without going into too much detail, SSDT gives us a command line utility called sqlpackage.exe. One thing we get to do with sqlpackage is create a DACPAC. A DACPAC is a representation of our database schema. Another thing we can do with sqlpackage is compare database schemas and dacpacs, pretty cool. There'e a whole bunch of other things you can do with it, but I only used the export, script and deployreport parameters for this little script. If you want to read more about sqlpackage please go <a href="https://msdn.microsoft.com/en-us/hh550080(v=vs.103).aspx" target="_blank">https://msdn.microsoft.com/en-us/hh550080(v=vs.103).aspx</a>. Microsoft tells it better than I can.

So this is the PowerShell script I used to do the schema compares.
 
```powershell
<# 
.SYNOPSIS
 Compare 2 databases using SSDT (sqlpackage.exe)

.DESCRIPTION
 Invoke an SSDT dewployment using sqlpackage.exe. This will generate a DACPAC for the source
 and then create a T-SQL Script and a DeployReport against a target database.

.PARAMETER SourceServer 
 Source database server. This is where the "gold copy" lives.

.PARAMETER SourceDatabase 
 Source database or the "gold copy" database. 

.PARAMETER TargetServer 
 Target database server. 

.PARAMETER TargetDatabase 
 Target database. This is the database that is being compared to the gold copy.

.PARAMETER ArtifactPath 
 Working directory for the scripts to be placed in

.PARAMETER SqlPackagePath 
 Path to sqlpackage.exe

.EXAMPLE
 Compare-DatabasesSSDT

#>
function Compare-DatabasesSSDT
{
    [cmdletbinding()]
    param(
        [parameter(Mandatory=$true)][string]$SourceServer, 
        [parameter(Mandatory=$true)][string]$SourceDatabase, 
        [parameter(Mandatory=$true)][string]$TargetServer, 
        [parameter(Mandatory=$true)][string]$TargetDatabase,
        [parameter(Mandatory=$true)][string]$ArtifactPath = 'c:\temp\', # A wowrking directory. Defaults to c:\temp
        [parameter(Mandatory=$false)][string]$SqlPackagePath = 'C:\Program Files\Microsoft SQL Server\120\DAC\bin\sqlpackage.exe' # Path to sqlpackage.exe
    )

    # Create artifact file if one does not exist.
    if (!(Test-Path -Path $($ArtifactPath)))
    {
        New-Item -ItemType Directory -Force -Path "$($ArtifactPath)" | Out-Null
    }

    try {
        $SourceDACPAC = "$($ArtifactPath)\$($SourceDatabase).dacpac"
        $TargetScript = "$($ArtifactPath)\$($TargetDatabase)_Script.sql"
        $TargetDriftReport = "$($ArtifactPath)\$($TargetDatabase)_DeployReport.xml"

        # sqlpackage export the source DACPAC
        &amp;"$SqlPackagePath" /a:Extract /ssn:$SourceServer /sdn:$SourceDatabase /tf:$SourceDACPAC /p:IgnorePermissions=True 2>&amp;1

        # sqlpackage script
        &amp;"$SqlPackagePath" /a:Script /sf:$SourceDACPAC /tsn:$TargetServer /tdn:$TargetDatabase /op:$TargetScript `
            /p:IgnorePermissions=True /p:IgnoreRoleMembership=True /p:IgnoreUserSettingsObjects=True 2>&amp;1

        # sqlpackage driftreport
        &amp;"$SqlPackagePath" /a:DeployReport /sf:$SourceDACPAC /tsn:$TargetServer /tdn:$TargetDatabase /op:$TargetDriftReport `
            /p:IgnorePermissions=True /p:IgnoreRoleMembership=True /p:IgnoreUserSettingsObjects=True 2>&amp;1

    }
    catch {
        throw $_
    }
}

# Unit testing commands
$ErrorActionPreference = "Stop"
$error.clear()

$params = @{
    'SourceServer' = 'localhost' ;
    'SourceDatabase' = 'AdventureWorks2012' ;
    'TargetServer' = 'localhost' ;
    'TargetDatabase' = 'AdventureWorks2012_2' ;
    'ArtifactPath' = 'c:\temp' ;
    'SqlPackagePath' = 'C:\Program Files\Microsoft SQL Server\120\DAC\bin\sqlpackage.exe'
}

Get-Date
Compare-DatabasesSSDT @params -Verbose 
Get-Date
#>
```

When I run it I wind up with 3 files in my ArtifactPath.

<img alt='' class='alignnone size-full wp-image-1303 ' src='/img/2016/06/img_576c0f9a49366.png' />

<strong>AdventureWorks2012.dacpac</strong> is the DACPAC created by the export command inside sqlpackage. It contains the schema for my AdventureWorks2012 database.

<strong>AdventureWorks2012_2_Script.sql</strong> contains the T-SQL code that would be run to bring AdventureWorks2012_2 to match AdventureWorks2012. That's pretty cool stuff.
 
```sql
/*
Deployment script for AdventureWorks2012_2

This code was generated by a tool.
Changes to this file may cause incorrect behavior and will be lost if
the code is regenerated.
*/

GO
SET ANSI_NULLS, ANSI_PADDING, ANSI_WARNINGS, ARITHABORT, CONCAT_NULL_YIELDS_NULL, QUOTED_IDENTIFIER ON;

SET NUMERIC_ROUNDABORT OFF;


GO
:setvar DatabaseName "AdventureWorks2012_2"
:setvar DefaultFilePrefix "AdventureWorks2012_2"
:setvar DefaultDataPath "C:\SQLData1\"
:setvar DefaultLogPath "C:\SQLLog1\"

GO
:on error exit
GO
/*
Detect SQLCMD mode and disable script execution if SQLCMD mode is not supported.
To re-enable the script after enabling SQLCMD mode, execute the following:
SET NOEXEC OFF; 
*/
:setvar __IsSqlCmdEnabled "True"
GO
IF N'$(__IsSqlCmdEnabled)' NOT LIKE N'True'
    BEGIN
        PRINT N'SQLCMD mode must be enabled to successfully execute this script.';
        SET NOEXEC ON;
    END
GO
USE [$(DatabaseName)];
GO

PRINT N'Update complete.';
GO
```

And finally <strong>AdventureWorks2012_2_DeployReport.xml</strong> contains an XML version of the changes that would be made to AdventureWorks2012_2 to bring it up to snuff with AdventureWorks2012.

```sql
<?xml version="1.0" encoding="utf-8"?>
<DeploymentReport xmlns="http://schemas.microsoft.com/sqlserver/dac/DeployReport/2012/02"><Alerts />
</DeploymentReport>
```

Based on the output of the T-SQL and the DeployReport XML file you can see there are no changes to be made to AdventureWorks2012_2. This is what I would expect as I just restored it from a fresh backup of the original AdventureWorks2012 database.

So just for giggles I created a new table in AdventureWorks2012 called SQLPackageDemo. Let me go ahead and run the script again and see what we get. And now you will find that new table in both the T-SQL script and the XML file. 

```sql
/*
Deployment script for AdventureWorks2012_2

This code was generated by a tool.
Changes to this file may cause incorrect behavior and will be lost if
the code is regenerated.
*/

GO
SET ANSI_NULLS, ANSI_PADDING, ANSI_WARNINGS, ARITHABORT, CONCAT_NULL_YIELDS_NULL, QUOTED_IDENTIFIER ON;

SET NUMERIC_ROUNDABORT OFF;


GO
:setvar DatabaseName "AdventureWorks2012_2"
:setvar DefaultFilePrefix "AdventureWorks2012_2"
:setvar DefaultDataPath "C:\SQLData1\"
:setvar DefaultLogPath "C:\SQLLog1\"

GO
:on error exit
GO
/*
Detect SQLCMD mode and disable script execution if SQLCMD mode is not supported.
To re-enable the script after enabling SQLCMD mode, execute the following:
SET NOEXEC OFF; 
*/
:setvar __IsSqlCmdEnabled "True"
GO
IF N'$(__IsSqlCmdEnabled)' NOT LIKE N'True'
    BEGIN
        PRINT N'SQLCMD mode must be enabled to successfully execute this script.';
        SET NOEXEC ON;
    END


GO
USE [$(DatabaseName)];


GO
PRINT N'Creating [dbo].[SQLPackageDemo]...';


GO
CREATE TABLE [dbo].[SQLPackageDemo] (
    [id]             INT        NOT NULL,
    [sqlpackagedemo] NCHAR (10) NOT NULL
);


GO
PRINT N'Update complete.';


GO

```

```sql
<?xml version="1.0" encoding="utf-8"?>
<DeploymentReport xmlns="http://schemas.microsoft.com/sqlserver/dac/DeployReport/2012/02">
<Alerts />
<Operations>
	<Operation Name="Create">
		<Item Value="[dbo].[SQLPackageDemo]" Type="SqlTable" />
	</Operation>
</Operations>
</DeploymentReport>
```

I personally like the output of the XML file better, especially if you're just planning on doing a compare for objects. I'm sure there are enhancements that could be made, and it still need to get wrapped in some sort of loop, but hopefully this is enough to get you started comparing your 57+ schemas. Once again than you for giving me a seed. Happy scripting everyone!

Disclaimer: Wanted to get this out there fairly quickly for the original requester. I will go back and do some editing at some point so if you find heinous abuse of the English language please forgive me. 
