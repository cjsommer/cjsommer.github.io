---
layout: post
title: High Performing Teams
date: 2023-04-20 20:00
author: cjsommer@gmail.com
comments: true
categories: [About Me]
---

What are some of the traits and guiding principles that we followed to make us a high performing team?

## Present a consistent image

Present a unified front.

## Check your ego at the door

There's no room for egos and heroes. Once politics enters the team it becomes a miserable place to live. 

## Adopt a "code first" mindset a.k.a. Automate Everything!

And I mean everything. Not much we did on that team was performed manually. Back when I started this thing called database administration it was really as simple as scripting everything. Working on AIX made ksh our primary automation tool and we used an enterprise scheduler for all of our orchestration.

Automating things meant we could downshift some of the support work to our operations team that was staffed 24/7/365. Rerunning failed backup jobs, rotating tapes out of the tape library, replacing disk drives, rebooting machines, just to name a few. The tools we built were fairly complex in some cases. 

## Build tools, don't just do tasks

This was a big part of the "code first" mindset from above. Every project was approached this way because they always produced something reusable. Maybe you wouldn't need it right away but if you follow this mindset you'll end up with a pile of tools in your toolbox. 

## Document, Document, Document

Design documentation is helpful for reviewing what you want to do with the team, as well as looking back to see why something might be written the way it is.

Support documentation is a requirement for downshifting. Nobody is going to accept the operations responsibilities of a script or a job without a good run guide.

## Start small

Automation can be as simple as a 2 line script that does something or it can be something that is very complex and requires thousands of lines of code. Some things cross system boundaries and require some sort of larger orchestration as well. 

If you decide to tackle a larger process it can be very daunting to think about all at once. Start small and eat the elephant one bite at a time.Break down that process down into smaller tasks and start at the most impactful step. And once you're done with that go onto the next step, and then the next, and so on. And before you know it your behemoth of a process will be automated and fully orchestrated.

## Own it. We built it, we own it, we support it.  

a.k.a. No throwing it over the wall. Nobody want to take ownership of a tool or project that someone else built. If you write the tool you own the tool. Someone else like a NOC might help you support the tool, but you are the owner. If it breaks its on you to fix it. 



This was 25 years ago and looking back I feel that was the highest performing team I'd ever been a part of.

Fast forward to today and I feel like we're drowning in automation tooling. Configuration management tools like Ansible, Puppet, Chef, Saltstack. Orchestration tools such as Control-M, Jenkins, Octopus Deploy, Nomad. Add in github, gitlab, atlassian, for source control. It's gotten a lot more complicated and I'm not really sure it's made it better. Honestly I feel all of the choice we have today has made it worse. 





What are the traits of a high performing team? 

[Definition of trait](https://www.merriam-webster.com/dictionary/trait) : a distinguishing quality (as of personal character).

What doesn't build a high performing team?
- Silos
- Independence
- Egos
- Heroes

So what does build a high performing team?
- Diverse in their specialties
- Diverse in their technical abilities
- Respectful of their peers regardless of title or current skill level
- Nurturing and patient
- Willing to share
- Willing to learn
- Autonomous
- Working toward a common goal
- Presenting a unified front
- A common mindset
    - Building tools, not doing tasks