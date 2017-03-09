---
layout: post
title: SQL Browser, what is it good for? Absolutely something!
date: 2017-03-08 10:00
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
Another great teaching opportunity landed in my lap this week. I got an email from a coworker looking for some help troubleshooting a SQL connection issue. He had an application server that could not connect to one of our SQL Servers.

We have a fairly complex and secure network environment. Multiple networks across multiple data centers with multiple firewalls in between. Because of this, one of the first things I typically look at is connectivity between application and database servers. The following question/answer session is how I attacked this particular problem, which led me to the actual issue.

*Q: Can you connect to the TCP port that SQL Server is listening on?*
A: Yes, in this case we could.

*Q: What does the connection string look like?*
A: The connection string was "Server=earth\ourplanet; Database='master'; Integrated Security=SSPI;"

*Q: Can the application server connect to UDP 1434 on the database server?*
A: Uhhh, nope.

Bingo. The application server could not connect to the SQL Browser service on UDP 1434. So maybe now you're asking why, and that's kinda the gist of this post. The SQL Browser provides a valuable service when an application tries to connect to a SQL Server named instance. The SQL Browser listens on UDP 1434 and provides information about all SQL Server instances that are installed on the server. One of those pieces of information is the TCP port number that SQL is listening on. Without that info, the application has no idea how to reach to your SQL Server, and will fail to connect. This was our exact issue.

I think it will make more sense if I show you what information the SQL Browser provides. Below is a PowerShell script that you can use to poll the SQL Browser service and below that is an example of the output.

```powershell
<#
 .Notes
 NAME: Get-SQLBrowserResponse.ps1
 AUTHOR: http://www.sqlservercentral.com/blogs/sqlmanofmystery/2013/02/27/finding-sql-server-installs-using-powershell/
 LASTEDIT:
 5/23/2012 - CJS - Initial Release
 4/4/2013 - CJS - Added better error handling template in the catch block

 .Synopsis
 Get all SQL Server instances from the SQL Browser using a UDP port probe

 .Description
 Get all SQL Server instances from the SQL Browser using a UDP port probe. SQL Browser service must be running or this script
 will not return anything.

 .Parameter Computer
 Computer name to probe

 .Example
 .\Get-SQLBrowserResponse.ps1 -Computer servername

 .LINK
 http://www.sqlservercentral.com/blogs/sqlmanofmystery/2013/02/27/finding-sql-server-installs-using-powershell/

#>
[cmdletbinding(
	DefaultParameterSetName = '',
	ConfirmImpact = 'low'
)]
Param(
	[Parameter(
		Mandatory = $True,
		Position = 0,
		ParameterSetName = '',
		ValueFromPipeline = $True)]
	[string]$Computer
)
Begin {
	$ErrorActionPreference = "SilentlyContinue";
	$Port = 1434
	$ConnectionTimeout = 1000
	$Responses  = @();
}
Process {
    # Determine IP and Hostname. This block will determine if it's a valid IP address automatically.
    # This allows us to pass in an IP address or a hostname to this script
    [System.Net.IPAddress]$IPAddressObject = $null

    if([System.Net.IPAddress]::tryparse($Computer,[ref]$IPAddressObject) -and $Computer -eq $IPAddressObject.tostring()) {
        $IPaddress = $Computer
        $hostinfo = [System.Net.Dns]::GetHostByAddress($IPaddress)
        $HostName = $($hostinfo.HostName.split('.'))[0].ToUpper()
    } else {
        $IPaddress = [System.Net.Dns]::GetHostAddresses($Computer)
        $Hostname = $Computer
    }

	$UDPClient = new-Object system.Net.Sockets.Udpclient
	$UDPClient.client.ReceiveTimeout = $ConnectionTimeout
	$UDPClient.Connect($IPAddress,$Port)
	$ToASCII = new-object system.text.asciiencoding
	$UDPPacket = 0x02,0x00,0x00
	Try {
		$UDPEndpoint = New-Object system.net.ipendpoint([system.net.ipaddress]::Any,0)
		$UDPClient.Client.Blocking = $True
		[void]$UDPClient.Send($UDPPacket,($UDPPacket.length))
		$BytesRecived = $UDPClient.Receive([ref]$UDPEndpoint)
		[string]$Response = $ToASCII.GetString($BytesRecived)
		$res = ""
		If ($Response) {
			$Response = $Response.Substring(3,($Response.Length-3)).Replace(";;","~")

			$Response.Split("~") | ForEach {
			$Responses += $_
			}
			$socket = $null;
			$UDPClient.close()
		}
	}
	Catch {
		$Error[0].ToString()
		$UDPClient.Close()
	}
}
End {
	return ,$Responses
}
```

```powershell
C:\Users\cjsommer\Documents\WindowsPowerShell [master +1 ~2 -1 !]> .\Get-SQLBrowserResponse.ps1 pluto

ServerName;PLUTO;InstanceName;NOTAPLANET;IsClustered;No;Version;11.0.5058.0;tcp;5150

C:\Users\cjsommer\Documents\WindowsPowerShell [master +1 ~2 -1 !]>
```

You can see from the output that the SQL Browser service provides server name, instance name, if it's clustered, SQL version, protocol and port. This server had only 1 instance on it, but if there were multiple instances they would all show up sequentially.

As I mentioned earlier, the SQL Browser service has to be accessible from the client if you want to connect to a named instance using instance name only. Just to prove it I ran a few test cases. Another thing to note, if you connect to a SQL Server by specifying the SQL Server listener port in the connection string, the SQL Browser is no longer required. I'll show you both of these conditions in the tests below.

Here is the PowerShell script I used for my tests.

```powershell
function Test-SQLBrowserConnection
{
    [cmdletbinding()]
    Param(
        [string]$Computer ,
        [int]$Port = 1434 # Default to SQL Browser service port
    )
    Begin {
        $ErrorActionPreference = "SilentlyContinue";
        $ConnectionTimeout = 1000
        $Responses  = @();
    }
    Process {
        # Determine IP and Hostname. This block will determine if it's a valid IP address automatically.
        # This allows us to pass in an IP address or a hostname to this script
        [System.Net.IPAddress]$IPAddressObject = $null

        if([System.Net.IPAddress]::tryparse($Computer,[ref]$IPAddressObject) -and $Computer -eq $IPAddressObject.tostring()) {
            $IPaddress = $Computer
            $hostinfo = [System.Net.Dns]::GetHostByAddress($IPaddress)
            $HostName = $($hostinfo.HostName.split('.'))[0].ToUpper()
        } else {
            $IPaddress = [System.Net.Dns]::GetHostAddresses($Computer)
            $Hostname = $Computer
        }

        $UDPClient = new-Object system.Net.Sockets.Udpclient
        $UDPClient.client.ReceiveTimeout = $ConnectionTimeout
        $UDPClient.Connect($IPAddress,$Port)
        $ToASCII = new-object system.text.asciiencoding
        $UDPPacket = 0x02,0x00,0x00
        Try {
            $UDPEndpoint = New-Object system.net.ipendpoint([system.net.ipaddress]::Any,0)
            $UDPClient.Client.Blocking = $True
            [void]$UDPClient.Send($UDPPacket,($UDPPacket.length))
            $BytesRecived = $UDPClient.Receive([ref]$UDPEndpoint)
            [string]$Response = $ToASCII.GetString($BytesRecived)

            If ($Response) {
                Write-Host "Connection to SQLBrowser Service on '$Computer' successful" -ForegroundColor Green
                $UDPClient.close()
            }
        }
        Catch {
            Write-Host "Connection to SQLBrowser Service on '$Computer' failed" -ForegroundColor Red
            $UDPClient.Close()
        }
    }
}

function Test-SqlConnection {
    [cmdletbinding()]
    param (
        $SQLServer
    )

    $SqlConnection = $null
    $SqlConnection = New-Object "System.Data.SqlClient.SQLConnection"
    $SqlConnection.ConnectionString = "Server=${SQLServer};Database='master';Integrated Security=SSPI; Connect Timeout = 1;"
    try {
        $SqlConnection.Open()
        Write-Host "Connection to SQL Server Instance '$SQLServer' successful" -ForegroundColor Green
    } catch {
        # Throw will generate a terminating error at this point
        Write-Host "Connection to SQL Server Instance '$SQLServer' failed" -ForegroundColor Red
    }
}
```

<hr>
<h2>Test Case #1</h2>
Configuration for this test case:
SQL Browser service is running and accessible on UDP 1434
SQL Server listening on default port TCP 1433
SQL Server named instance

```powershell

Write-Host "Test Case #1"
Test-SQLBrowserConnection -Computer mercury # Test SQL Browser service
Test-SqlConnection -SQLServer 'mercury\firstplanet' # Test instance name only
Test-SqlConnection -SQLServer 'mercury,1433' # Test SQL port number
Write-Host ""

Test Case #1
Connection to SQLBrowser Service on 'mercury' successful
Connection to SQL Server Instance 'mercury\firstplanet' successful
Connection to SQL Server Instance 'mercury,1433' successful

```

Results of this test tell me in this configuration there are no issues connecting to your SQL Server using instance name or SQL port number.

<hr>
<h2>Test Case #2</h2>
Configuration for this test case:
SQL Browser service is not running and therefor inaccessible
SQL Server listening on default port TCP 1433
SQL Server named instance

```powershell

Write-Host "Test Case #2"
Test-SQLBrowserConnection -Computer earth # Test SQL Browser service
Test-SqlConnection -SQLServer 'earth\ourplanet' # Test instance name only
Test-SqlConnection -SQLServer 'earth,1433' # Test SQL port number
Write-Host ""

Test Case #2
Connection to SQLBrowser Service on 'earth' failed
Connection to SQL Server Instance 'earth\ourplanet' failed
Connection to SQL Server Instance 'earth,1433' successful

```

The results for this test are slightly more interesting. When trying to connect using the instance name only, the connection fails. This can be attributed to the fact that SQL Browser service is not running. Connecting to a named instance requires the SQL Browser to be running.

The other thing to note is that the connection explicitly using SQL port 1433 is successful.

<hr>
<h2>Test Case #3</h2>
Configuration for this test case:
SQL Browser service is not running and therefore inaccessible
SQL Server listening on non-default port TCP 5150
SQL Server named instance

```powershell

Write-Host "Test Case #3"
Test-SQLBrowserConnection -Computer pluto # Test SQL Browser service
Test-SqlConnection -SQLServer 'pluto\notaplanet' # Test instance name only
Test-SqlConnection -SQLServer 'pluto,5150' # Test SQL port number
Write-Host ""

Connection to SQLBrowser Service on 'pluto' failed
Connection to SQL Server Instance 'pluto\notaplanet' failed
Connection to SQL Server Instance 'pluto,5150' successful

```
The results of this test are pretty much the same as test #2. The only thing I wanted to show here was a SQL Server using the non default port 5150.

<hr>
<h2>Summary</h2>
If you're connecting to a SQL Server named instance using instance name only, the SQL Browser service becomes an important piece of the connectivity puzzle. Without the SQL Browser all connections will fail. The only condition that always connected was by explicitly using the TCP port number in the connection string. That's because you are telling the application (in the connection string) that SQL Server is listening on a specific port number. Hopefully the examples above help explain it better than I can in a sentence, and hopefully this helps someone out in the future.


