---
layout: post
title: Identity Column Increment Value (EVEN/ODD)
date: 2015-11-17 22:11
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
<hr>
As always I'm sitting at my desk, minding my own business, when an email comes in from a developer.

<em>"Hey DBA's, I have a thought about a really creative solution to this really hard problem. I have two tables in my database and each has an identity column, each of which is currently incrementing by 1. I'd like to change each to increment by 2. Also, one of the tables will need to only contain ODD numbered values, and the other will need to only contain EVEN numbered values. This will help me solve my really hard problem."</em>

So the first question I had to answer was can you change the increment value of an identity column? Short answer, yes. Caveat, it requires you to rebuild the table to do so. Not a big deal.

I could have replied with that and sent my developer on his way, but I got to thinking about the other part of his question. Was the whole EVEN/ODD thing a good design idea? And this is the question I am trying to answer in this post.

So what I wanted to determine if there was a way to ensure that only ODD values were inserted into an identity column. Sounds simple enough, right? Create the table with a starting value of 1, with an increment value of 2. That should do exactly my developer needed...right?
<hr>
<h3>The IncrementTest Table Creation Script</h3>
So I created a table in mysandbox database just for this test.

```sql
USE [mysandbox] ;
/****** Object:  Table [dbo].[IncrementTest]    Script Date: 2015-11-11 19:12:29 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
SET ANSI_PADDING ON
GO

CREATE TABLE [dbo].[IncrementTest](
	[ID] [int] IDENTITY(1,2) NOT NULL,
	[TestData] [varchar](50) NOT NULL
) ON [PRIMARY] ;

SET ANSI_PADDING OFF
GO
```

<h3>T-SQL Script to Test the Inserts</h3>
Next I added a handful of records to the table just to see what the behavior of the identity column would be.

```sql
/****** Script for SelectTopNRows command from SSMS  ******/
TRUNCATE TABLE [mysandbox].[dbo].[IncrementTest];

-- Load up 10 records. Data doesnt matter, just want to see the identity column.
DECLARE @cnt INT = 0

WHILE @cnt &lt; 10
BEGIN
  INSERT INTO [mysandbox].[dbo].[IncrementTest] (TestData)
  VALUES ('Testdata');

  SET @cnt = @cnt + 1;
END

SELECT *
FROM [mysandbox].[dbo].[IncrementTest];
```

<h3>First Set of Results</h3>
And here are the results of the first batch of inserts. Sure enough you can see that the values in the table remain ODD and increment by 2, just like the developer wanted.

<img alt='' class='alignnone size-full wp-image-1129 ' src='/img/2015/11/img_564bd33e97c7e.png' />

<hr>
<h3>Insert my devil's advocate. How can I possibly break this?</h3>
<img alt='' class='alignright size-full wp-image-1138 ' src='/img/2015/11/img_564bd9d6a69ab.png' />
Enter, IDENTITY_INSERT! With IDENTITY_INSERT OFF (the default), the identity column is automatically incremented for you. IDENTITY_INSERT ON allows you to specify the value of an identity column manually upon insert. But will it allow me to change the ID's to EVEN values?

The following script determines the MAX value in the table (19), it adds 1 to that number (20), and then it inserts a few more records after that just to prove my point. Hey wait, 20 is an even number! Will that even work? Not sure. I'm trying to punch holes in the EVEN/ODD design that my developer wants to use, remember.

```sql
-- MAX ID Should be 19 at this point
DECLARE @maxid INT = (
    SELECT MAX(ID)
    FROM [mysandbox].[dbo].[IncrementTest]
    );

SET IDENTITY_INSERT [mysandbox].[dbo].[IncrementTest] ON;

INSERT INTO [mysandbox].[dbo].[IncrementTest] (
  ID,
  TestData
  )
VALUES (
  (@maxid + 1),
  'Testdata with identity insert on'
  );

SET IDENTITY_INSERT [mysandbox].[dbo].[IncrementTest] OFF;
-- Load up 10 records. Data doesn't matter, just want to see the identity column.
SET @cnt = 0;

WHILE @cnt &lt; 10
BEGIN
  INSERT INTO [mysandbox].[dbo].[IncrementTest] (TestData)
  VALUES ('Testdata');

  SET @cnt = @cnt + 1;
END

SELECT *
FROM [mysandbox].[dbo].[IncrementTest];
```

<h3>The Final Results</h3>
As you can see from the results below I was able to punch a hole into the EVEN/ODD design just by using IDENTITY_INSERT.
<img class="alignnone size-full wp-image-1118 " src="/img/2015/11/img_5643da58bf447.png" alt="" />

Is this a big deal? In my best DBA voice I think I have to give it an "it depends". If it's a temporary thing then you might be able to get away with it, but if it's going to be a permanent solution the potential risk of it happening goes up. How bad will it break the application if it is expecting EVEN values and it finds ODD ones? How well is this design documented and how well would it be understood by future developers? All questions you should really consider when coming up with creative solutions like this. It's always better to find the gaps in the design before it goes into production.

<hr>
<strong>+Update:</strong> @EdDebug asked me on twitter
<a href="https://twitter.com/EdDebug/status/666867897839562752" target="_blank"><img alt='' class='alignnone size-full wp-image-1147 ' src='/img/2015/11/img_564c78e3d0748.png' /></a>
Hmm, interesting idea. I also considered trying triggers. I think this post will see a part 2 to see if I can overcome the EVEN/ODD issue I found above.
<hr>
