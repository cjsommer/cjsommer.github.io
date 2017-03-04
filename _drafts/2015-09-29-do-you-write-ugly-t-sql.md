---
layout: post
title: Do you write ugly T-SQL?
date: 2015-09-29 09:00
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
Are your scripts littered with commented blocks of T-SQL code?
<pre class="theme:ssms2012 lang:tsql decode:true " title="Random blocks of code commented out" >
select fname,lname 
from usertable
where id=1
--and lname like 'smith%'
/*
-- Who knows why this block is commented out
select fname,lname 
from usertable
where id=1
*/</pre> 

Do you just start typing and rely on word wrap to fit your query into the SSMS query window?
<pre class="theme:ssms2012 lang:tsql decode:true " title="Unformatted run-on TSQL" >
select users.fname,users.lname,users.address,users.city,users.state,users.zip,
users.phone,users.email from users inner join managers on managers.id = 
users.id where id=1 and manager.lname = 'smith'
</pre>

Do you omit whitespace in your queries in an effort to save space?
<pre class="theme:ssms2012 lang:tsql decode:true " title="Lack of whitespace" >
SELECT fname,lname FROM users WHERE lname='smith' 
</pre>
<hr>
If any or all of those cases define your scripts then yes, I may just be talking to you. 

"Chris, Surely you jest!" No I am being totally serious, and stop calling me Shirley! I will admit that this sounds nitpicky, but chances are many of us haven't even thought about how our code looks and why it might matter. We're just happy that it runs most of the time, right? 

Consistent formatting enhances the readability of your code. If you are part of a team, there will come a time when you have to support each other's code. There is also a good chance that you will write a piece of code and not have to look at it again for years. If you are a solo DBA you may think that formatting doesn't matter because you are the only one who has to support it, but you couldn't be more wrong. I will say it again, consistent formatting enhances the readability of your code. Developers have known this and been following formatting standards for quite some time. It's typically a part of their peer review processes because it's that important to them. 

Take these 2 examples of the exact same T-SQL code. I promise you they are identical except for the formatting.

<pre class="theme:ssms2012 lang:tsql decode:true " title="Unformatted T-SQL" >
select case
when ( exists (
select 1 AS [C1] from [dbo].[MyTable] AS [Extent1]
WHERE (((UPPER([Extent1].[Column1])) = (UPPER(@Column1))) OR ((UPPER([Extent1].[Column1]) IS NULL)
AND (UPPER(@Column1) IS NULL)
))AND (([Extent1].[Column2] = @Column2)
OR (
([Extent1].[Column2] IS NULL) AND (@Column2 IS NULL)
)or ([Extent1].[Column2] = @Column3) OR (([Extent1].[Column2] IS NULL)
and (@Column3 IS NULL)
)
)AND ([Extent1].[Column4] <> @Column4)))
then cast(1 AS BIT) else cast(0 AS BIT)
end AS [C1]
FROM (
SELECT 1 AS X
) AS [SingleRowTable1]
</pre>

<pre class="theme:ssms2012 lang:tsql decode:true " title="Formatted T-SQL using http://poorsql.com/" >
SELECT CASE 
  WHEN (
    EXISTS (
     SELECT 1 AS [C1]
     FROM [dbo].[mytable] AS [Extent1]
     WHERE (
       ((Upper([Extent1].[column1])) = (Upper(@Column1)))
       OR (
        (Upper([Extent1].[column1]) IS NULL)
        AND (Upper(@Column1) IS NULL)
        )
       )
      AND (
       ([Extent1].[column2] = @Column2)
       OR (
        ([Extent1].[column2] IS NULL)
        AND (@Column2 IS NULL)
        )
       OR ([Extent1].[column2] = @Column3)
       OR (
        ([Extent1].[column2] IS NULL)
        AND (@Column3 IS NULL)
        )
       )
      AND ([Extent1].[column4] <> @Column4)
     )
    )
   THEN Cast(1 AS BIT)
  ELSE Cast(0 AS BIT)
  END AS [C1]
FROM (
 SELECT 1 AS X
 ) AS [SingleRowTable1]
</pre>
Which one would you prefer to read at 3:00 AM when the application is down?

Honestly the hardest part is getting the team to agree on the formatting standards (Commas before or after column names, Keywords upper or lower case, Indentation using tabs or spaces, Break join statements, etc.). Actual formatting of the code couldn't be simpler really. There are plenty of tools that do it for us. Here are a few examples (I use the poorsql plugin for SSMS and I love it).

Commercial Tool: <a href="http://www.red-gate.com/products/sql-development/sql-prompt/">http://www.red-gate.com/products/sql-development/sql-prompt/</a>
Free Tool with an SSMS plugin: <a href="http://poorsql.com/">http://poorsql.com/</a>
ApexSQL Refactor is another great free tool: <a href="https://www.apexsql.com/sql_tools_refactor.aspx">https://www.apexsql.com/sql_tools_refactor.aspx</a>
Online Tool: <a href="http://www.dpriver.com/pp/sqlformat.htm">http://www.dpriver.com/pp/sqlformat.htm</a>

So if you are not already using some sort of formatting standard for your T-SQL code I highly recommend you start. A consistent look and feel helps you understand what you are looking at more quickly, which is especially important at 3:00 AM. 
