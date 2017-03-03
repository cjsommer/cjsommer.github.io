---
layout: post
title: How the SQL Agent Job schedule_uid broke my heart, and my jobs!
date: 2017-02-03 12:15
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---
This is something that has caused me some grief in my life as a DBA. I hesitate to call it a bug, but this little gottcha resurfaced in a change that was submitted by a teammate just today. So I wanted to share while it was fresh in my mind.

When you script out a SQL Agent Job you'll notice that the job schedule will have a schedule_uid parameter (providing your job has a schedule). The gottcha lies in that schedule_uid. If you create another job schedule with the same schedule_uid, it will overwrite the schedule for any jobs that are using it. i.e. Any other jobs that are using that schedule_uid will start using the new schedule. Normally I consider UID's as very unique and chances of a collision are low, but if you do a fair amount of copying jobs between SQL Servers there's a good chance this will bite you eventually. That's what happened to us (more than once).

Lets see if I can explain it better in an example.

I have a job called 'Test Job'. The steps that make up 'Test Job' aren't important for this blog post. I'm more focused on the schedule for this job. The screenshot below shows that the schedule for 'Test Job' is set to run every 5 minutes. Agree? Good.

<img alt='' class='alignnone size-full wp-image-1540 ' src='/img/2017/02/img_5894b7b48e1b6.png' />

When I script out 'Test Job' in SSMS, this is what I get in the schedule section.

```sql
EXEC @ReturnCode = msdb.dbo.sp_add_jobschedule @job_id=@jobId, @name=N'5 Minutes',
		@enabled=1,
		@freq_type=4,
		@freq_interval=1,
		@freq_subday_type=4,
		@freq_subday_interval=5,
		@freq_relative_interval=0,
		@freq_recurrence_factor=0,
		@active_start_date=20170123,
		@active_end_date=99991231,
		@active_start_time=0,
		@active_end_time=235959,
		@schedule_uid=N'89ef7def-1c85-4791-80e6-fc494514e356'
```

So the problem lies in the schedule_uid. If you create another job schedule with the same schedule_uid, it will overwrite the schedule for any jobs that are using it. i.e. Any other jobs that are using that same schedule_uid will start using the new schedule.

Lets create a second job called 'My New Job'. Below is the T-SQL that was used to create the schedule for 'My New Job' and notice that it uses the same schedule_uid as 'Test Job'. Once again the actual job isn't important, I'm just talking about the job schedule itself.

```sql
EXEC @ReturnCode = msdb.dbo.sp_add_jobschedule @job_id=@jobId, @name=N'Daily',
		@enabled=1,
		@freq_type=4,
		@freq_interval=1,
		@freq_subday_type=1,
		@freq_subday_interval=0,
		@freq_relative_interval=0,
		@freq_recurrence_factor=0,
		@active_start_date=20170203,
		@active_end_date=99991231,
		@active_start_time=10000,
		@active_end_time=235959,
		@schedule_uid=N'89ef7def-1c85-4791-80e6-fc494514e356'
```

The job schedule for 'My New Job' is set to run daily at 1:00 AM. If I look at the schedule for 'My New Job', it looks fine. Agree?

<img alt='' class='alignnone size-full wp-image-1532 ' src='/img/2017/02/img_5894b01fcc2f9.png' />

But guess what else happened? Since I used the same schedule_uid in 'My New Job', it overwrote the schedule for 'Test Job' too.

<img alt='' class='alignnone size-full wp-image-1533 ' src='/img/2017/02/img_5894b05d52d70.png' />


I'm hesitant to call this a bug, but I would have thought that sp_add_jobschedule would fail because a schedule with a matching schedule_uid already exists, but that's not the case. It just seems to overwrite the schedule details and move along, leaving DBAs to clean up the mess.

How do you get around this issue? All you need to do is remove the schedule_uid from the sp_add_jobschedule step and it won't overwrite any of the existing job schedules. It will assign a new UID upon creation.

We found this because we copy SQL Agent Jobs between servers and between environments pretty frequently. We've had this happen a number of times where someone was creating or copying a job that included the schedule_uid, and it changed the schedules of a number of existing jobs. In our case it we eventually caught it, but in some cases not for months. Hopefully this saves someone the trouble of chasing this one down as we have done in the past. Good luck and happy scripting.
