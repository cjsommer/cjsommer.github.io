---
layout: post
title: Using PowerShell to Export SQL Data to CSV. How well does it perform?
date: 2015-04-28 08:15
author: cjsommer@gmail.com
comments: true
categories: [PowerShell, SQL Server]
---
<a href="/img/2015/04/ExportCSV-TwitterPost.jpg"><img class="alignright size-full wp-image-351" src="/img/2015/04/ExportCSV-TwitterPost.jpg" alt="ExportCSV-TwitterPost" width="266" height="269" /></a>

So here we are at week 4 of the #SQLNewBlogger challenge. Earlier in the week I responded to the following post on Twitter #sqlhelp, and after I posted my response I thought that this would make a fun blog post. I have used PowerShell to export SQL Server tables to CSV files before so I know that my suggestion works, but I was wondering if I could determine how performance would be for a larger table.
<h2>Test Details</h2>
<ul>
	<li>I will be exporting a SQL Server table to a CSV file.</li>
	<li>I will be starting with a row count of 1000 rows, and I will increase the row count incrementally to a maximum of 2,000,000 rows. </li>
	<li>I will take no fewer than 10 samples for each different row count.</li>
	<li>I will be logging the statistics of each run into a RunStats table so I can easily extrapolate the results once all the tests are done.</li>
</ul>
<h2>Test Platform</h2>
The test platform is my old laptop so nothing fancy. It shouldn't be hard to push this bad boy past its limits.

<a href="/img/2015/04/ExportCSV-TestPlatform.jpg"><img class="alignnone size-full wp-image-355" src="/img/2015/04/ExportCSV-TestPlatform.jpg" alt="ExportCSV-TestPlatform" width="560" height="163" /></a>
<ul>
	<li>Windows 7 Professional SP1</li>
	<li>SQL Server 2012</li>
	<li>PowerShell 4</li>
</ul>
I used <a href="http://www.databasetestdata.com/" target="_blank">www.databasetestdata.com</a> to generate a simple table and 1000 rows of test data to seed my test. It's a pretty simple table. No indexes or constraints or anything like that.
<pre class="theme:ssms2012 lang:tsql decode:true " title="Addresses Table">/****** Object:  Table [dbo].[addresses]    Script Date: 4/24/2015 10:07:15 PM ******/
CREATE TABLE [dbo].[addresses](
	[Email] [varchar](50) NULL,
	[Full Name] [varchar](50) NULL,
	[Country] [varchar](50) NULL,
	[User Id] [varchar](50) NULL,
	[Created At] [varchar](50) NULL
) ON [PRIMARY] ;
</pre>
The export is performed by piping the output of Invoke-SqlCmd to the Export-CSV cmdlet. Here is the script.
<pre class="lang:ps decode:true " title="Export-CSV-Testing.ps1">Push-Location; Import-Module SQLPS -DisableNameChecking; Pop-Location

$SQLServer = "localhost\inst1"
$DBName = "ExportCSVTesting"
$ExportFile = "C:\Users\BIGRED-7\Documents\Git\csvfiles\addresses.csv"
$Counter = 0

while ( $true )
{
    # Remove the export file
    if (Test-Path -Path $ExportFile -PathType Leaf) {
        Remove-Item $ExportFile -Force
    }

    # Clear the buffer cache to make sure each test is done the same
    $ClearCacheSQL = "DBCC DROPCLEANBUFFERS"
    Invoke-Sqlcmd -ServerInstance $SQLServer -Query $ClearCacheSQL

    # Export the table through the pipeline and capture the run time. Only the export is included in the run time.
    $ExportSQL = "SELECT * FROM [addresses] ;"

    $sw = [Diagnostics.Stopwatch]::StartNew()
    Invoke-Sqlcmd -ServerInstance $SQLServer -Database $DBName -Query $ExportSQL | Export-CSV -Path $ExportFile -NoTypeInformation
    $sw.Stop()
    $sw.Elapsed
    $Milliseconds = $sw.ElapsedMilliseconds

    # Get a row count for display
    $RowCountSQL = "SELECT COUNT(0) AS [Count] FROM [addresses] ;"
    $RowCount = Invoke-Sqlcmd -ServerInstance $SQLServer -Database $DBName -Query $RowCountSQL
    $RowCount = $RowCount.Count
    
    $Counter++
    Write-Output ("Run $Counter of RowCount: $RowCount")

    # Log the run statistics
    $StatsSQL = "INSERT INTO [RunStats] (Counter,Milliseconds,Notes) VALUES ($RowCount,$Milliseconds,'Pipeline')"
    Invoke-Sqlcmd -ServerInstance $SQLServer -Database $DBName -Query $StatsSQL
}

</pre>
So basically I fire up this test for each different row count and let it collect a bunch of run time statistics. I collected a minimum of 10 samples for each different row count. Storing the results of each export in the database worked out really well because it made it easy to calculate the results I was looking for.
<h2>Run Statistics</h2>
Here is the query I used to calculate the run statistics
<pre class="theme:ssms2012 lang:tsql decode:true" title="Run Statistics">SELECT Counter,
	COUNT(0) AS Samples
	,MIN(Milliseconds) AS ms_Min
	,AVG(Milliseconds) AS ms_Avg
	,MAX(Milliseconds) AS ms_Max
	,CAST(Counter / AVG(Milliseconds) AS decimal) AS rows_per_ms
  FROM [ExportCSVTesting].[dbo].[RunStats]
  GROUP BY Counter</pre>

<a href="/img/2015/04/ExportCSVStats.jpg"><img src="/img/2015/04/ExportCSVStats.jpg" alt="ExportCSVStats" width="451" height="228" class="alignright size-full wp-image-378" /></a>

...and here are the results. As you can see the number of rows exported didn't seem to affect the overall performance of the export. I only pushed it to 2,000,000 rows on my laptop because it ran out of memory resources after that, but I was surprised none the less. I really thought the rows_per_ms (number of rows exported per millisecond) might drop off but it never did. It consistently got 10-12 rows per millisecond not matter how big the table was.

One observation I made while it was running was that the PowerShell memory usage increased dramatically as I started adding rows to the export. At 2,000,000 rows my table is 187 MB in size, and the PowerShell memory usage climbed to nearly 1 GB during the export tests. I did try going for 10 million rows but I ended up getting an out of memory exception (which I didn't capture a screenshot of). So I guess I can safely say that if you are doing large exports, you will need plenty of memory for PowerShell as it appears to be very memory hungry using this process.  

<h2>Conclusion</h2>
I did not see the performance drop off as the table grew, but I did hit a brick wall with the limited memory on my machine so that would be something to be aware of. Based on the memory usage I saw I think it is safe to say that 64-bit PowerShell would be a requirement for larger tables.

I think I can safely say that PowerShell is still a decent option for exporting smaller tables to CSV. Is PowerShell the best option for exporting large amounts of data? Not sure as I didn't test any other methods and that really wasn't the purpose of this post. People had offered other suggestions in response to that twitter post like BCP and SSIS which definitely warrants some testing before I can make that determination (maybe for another blog post). 
