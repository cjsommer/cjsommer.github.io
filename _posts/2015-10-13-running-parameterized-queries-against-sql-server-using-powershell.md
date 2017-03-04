---
layout: post
title: Running Parameterized Queries against SQL Server using PowerShell
date: 2015-10-13 09:00
author: cjsommer@gmail.com
comments: true
categories: [PowerShell, SQL Server]
---
<hr>
For many years I didn't really think about the implications of how I was retrieving data from my SQL Servers in PowerShell. I was just happy that I was able to retrieve the data! As I learned more about SQL Server I started to think of things like SQL injection and using parameterized queries to promote plan reuse. 

I went back and looked at some of the old PowerShell scripts that I had written and found that I was way off! Most of the old scripts would be prime candidates for SQL Injection. I wasn't really concerned early on because I was the one running my scripts and passing in the parameters, but as my scripts became more automated and database driven, they became more vulnerable to SQL Injection. And parameterized queries? What the heck is a parameterized query? I had no clue when I first started out. 

This blog post is about how I changed my thinking and my methods.

<h3>A little about my test configuration</h3>
<a href="/img/2015/10/DeleteMeTables.jpg"><img src="/img/2015/10/DeleteMeTables.jpg" alt="DeleteMeTables" width="334" height="178" class="alignright size-full wp-image-998" /></a>

I am running PowerShell 4.0 along with SQL Server 2012 on my laptop, so nothing special there really. Each test script is a PowerShell function that performs a SELECT statement against [person].[person] table in the AdventureWorks2012 database. The function also accepts a parameter for 'LastName' which is used in the search predicate. Pretty straight forward and very common. You'll see the actual scripts below in all of the code snippets.

There is also a table in the AdventureWorks2012 database called DeleteMe. DeleteMe is used as the target of my SQL Injection attacks. The SQL Injection attack I try in all 3 examples is to try and delete that table.

Ultimately I wanted to find a method that protects me from SQL injection and promotes plan reuse by using parameteried queries. Enough about that, lets see the fun stuff!
<br  clear="all"/><hr>
<h3>Select-Unparameterized using SQLPS and Invoke-Sqlcmd</h3>
This script illustrates the way I first learned to run queries against SQL Server. I am using variable substitution to generate dynamic SQL, and then running it using Invoke-Sqlcmd. I also output the dynamic SQL to the screen so we can see what it is going to do.  

```powershell
function Select-UnparameterizedSQLPS
{
    [cmdletbinding()]
    param (
        [string]$LastName
    )
    Push-Location; Import-Module SQLPS -DisableNameChecking; Pop-Location
    $sql1 = "
    SELECT [BusinessEntityID]
      ,[PersonType]
      ,[NameStyle]
      ,[Title]
      ,[FirstName]
      ,[MiddleName]
      ,[LastName]
      ,[Suffix]
      ,[EmailPromotion]
      ,[AdditionalContactInfo]
      ,[Demographics]
      ,[rowguid]
      ,[ModifiedDate]
      FROM [AdventureWorks2012].[Person].[Person]
      WHERE [LastName] = '${LastName}'
      "
    $sql1
    Invoke-Sqlcmd -ServerInstance 'localhost\inst1' -Database 'AdventureWorks2012' -Query $sql1
}

Clear-Host
$result = Select-UnparameterizedSQLPS -LastName "Duffy' ; DROP TABLE [DeleteMe] ;--"
$result | Format-Table -AutoSize
```

The SQL Query:
<a href="/img/2015/10/Select-UnparameterizedSQLPS.jpg"><img src="/img/2015/10/Select-UnparameterizedSQLPS.jpg" alt="Select-UnparameterizedSQLPS" width="465" height="224" class="alignnone size-full wp-image-1001" /></a>

<a href="/img/2015/10/DeleteMe_Missing.jpg"><img src="/img/2015/10/DeleteMe_Missing.jpg" alt="DeleteMe_Missing" width="331" height="181" class="alignright size-full wp-image-997" /></a>

It is clear from the SQL output what this was going to do, and sure enough, the DeleteMe table is now gone. Not only is this query vulnerable to SQL injection, but it also does not promote plan reuse because the T-SQL includes the literal value in the search predicate. 
<br  clear="all"/><hr>
<h3>Select-Parameterized using SQLPS and Invoke-Sqlcmd</h3>
The second method I tried was by using Invoke-Sqlcmd and using sp_executesql to run my query as a parameterized query. The parameterization did help with plan reuse, but it did not prevent against SQL Injection attack like I thought it would.

```powershell
function Select-ParameterizedSQLPS
{
    [cmdletbinding()]
    param (
        [string]$LastName
    )
    Push-Location; Import-Module SQLPS -DisableNameChecking; Pop-Location
    $sql1 = "N'SELECT [BusinessEntityID]
      ,[PersonType]
      ,[NameStyle]
      ,[Title]
      ,[FirstName]
      ,[MiddleName]
      ,[LastName]
      ,[Suffix]
      ,[EmailPromotion]
      ,[AdditionalContactInfo]
      ,[Demographics]
      ,[rowguid]
      ,[ModifiedDate]
    FROM [AdventureWorks2012].[Person].[Person] 
    WHERE [LastName] = @LastName'"
    $sql1
    $Params = "N'@LastName VARCHAR(50)'"
    $Query = "EXECUTE sp_executesql @stmt = $sql1, @params = $Params, @LastName = $LastName ;"

    Invoke-Sqlcmd -ServerInstance 'localhost\inst1' -Database 'AdventureWorks2012' -Query $Query
}

Clear-Host
$result = Select-ParameterizedSQLPS -LastName "'Duffy' ; DROP TABLE [DeleteMe] ;--"
$result | Format-Table -AutoSize
```

The SQL Query:
<a href="/img/2015/10/Select-ParameterizedSQLPS.jpg"><img src="/img/2015/10/Select-ParameterizedSQLPS.jpg" alt="Select-ParameterizedSQLPS" width="405" height="210" class="alignnone size-full wp-image-1000" /></a>

<a href="/img/2015/10/DeleteMe_Missing.jpg"><img src="/img/2015/10/DeleteMe_Missing.jpg" alt="DeleteMe_Missing" width="331" height="181" class="alignright size-full wp-image-997" /></a>

It was not clear from the raw T-SQL output what this query was going to do because it was parameterized. This was just one version of how I tried to use sp_executesql to get around the injection attack, but as I found out there really is no good way to do it using Invoke-Sqlcmd. After running this script my DeleteMe table was gone. This appears to be a huge limitation in Invoke-Sqlcmd for running ad-hoc queries.

<br  clear="all"/><hr>
<h3>Select-Parameterized using ADOLib and Invoke-Query</h3>
So for my third attempt I thought I would take a different approach and use straight .NET, which is when I also found the SQLPSX module. SQLPSX is actually a bundle of other modules and it includes the adoLib module. adoLib is a really nice because uses straight .NET under the hood. It's fast, it's efficient, and in this case it's just what the doctor ordered.

```powershell
function Select-ParameterizedADOLib
{
    [cmdletbinding()]
    param (
        [string]$LastName
    )
    Import-Module adoLib
    $sql1 = 'SELECT [BusinessEntityID]
        ,[PersonType]
        ,[NameStyle]
        ,[Title]
        ,[FirstName]
        ,[MiddleName]
        ,[LastName]
        ,[Suffix]
        ,[EmailPromotion]
        ,[AdditionalContactInfo]
        ,[Demographics]
        ,[rowguid]
        ,[ModifiedDate]
    FROM [AdventureWorks2012].[Person].[Person] 
    WHERE [LastName] = @LastName'
    $sql1
    $params = @{'LastName'=$LastName} 
    $conn = New-Connection 'localhost\inst1' -database 'AdventureWorks2012'
    Invoke-Query -connection $conn -sql $sql1 -parameters $params
}

Clear-Host
$result = Select-ParameterizedADOLib -LastName "Duffy' ; DROP TABLE [DeleteMe] ;--"
$result | Format-Table -AutoSize
```

The SQL Query:
<a href="/img/2015/10/Select-ParameterizedADOLib.jpg"><img src="/img/2015/10/Select-ParameterizedADOLib.jpg" alt="Select-ParameterizedADOLib" width="389" height="238" class="alignnone size-full wp-image-999" /></a>

<a href="/img/2015/10/DeleteMeTables.jpg"><img src="/img/2015/10/DeleteMeTables.jpg" alt="DeleteMeTables" width="334" height="178" class="alignright size-full wp-image-998" /></a>
Once again it isn't really clear what would happen based on the raw T-SQL that was output from the function, but after running the query I did find that my DeleteMe table remained intact. This implementation protected me from that SQL injection and because it is a parameterized query it also helps with plan reuse.
<br  clear="all"/><hr>
<h3>Conclusion</h3>
Like with most thing PowerShell there are a lot of different ways to accomplish your task. The key to finding the best method is in trying to understand the behavior and trying to break it. I love breaking stuff! In this case I tried to break my functions by passing in bogus parameters that included a DROP TABLE statement. In 2 out of the 3 functions I found that I was vulnerable to SQL injection and able to drop that table. 

You won't start out knowing the best methods for accomplishing a task and that's OK. Don't let that stop you from picking up the keyboard and banging out some code. Just understand that you will find better ways of doing things over time, and that's a good thing! Embrace change and use that additional knowledge to become better at what you do.
