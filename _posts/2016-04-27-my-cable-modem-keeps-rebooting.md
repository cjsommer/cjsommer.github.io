---
layout: post
title: My Cable Modem Keeps Rebooting!
date: 2016-04-27 10:00
author: cjsommer@gmail.com
comments: true
categories: [Tech]
---
There are obviously quite a few things that could cause internet issues that are totally out of our control, but there are some things that we may be able to try to help ourselves. Face it, dealing with cable service calls is right up there with doing your taxes. If you are a cable internet subscriber and are having stability problems this post may help you solve your issues without having to call the cable company.  

A couple of weeks ago I was working from home and my kids were off from school, so needless to say we had a couple devices using our internet connection. One afternoon that week my internet started cycling. It would go down for about 2-3 minutes and then recover. If it were a one time occurrence I probably would have ignored it, but it was bouncing every 30 minutes and it made the "working" part of working from home nearly impossible. I knew I had to troubleshoot.

My first thought was to call my internet provider, but you know how those 1st support level calls go. They give you some boiler plate answer, have you reboot all the things and then tell you to call back if it keeps happening. I wanted to do a little digging so I could provide them with some evidence/data when I finally called. So sure enough within 30 minutes my internet went down again and I started to investigate. One of the first things I noticed was that the lights on my cable modem were cycling as if it had just rebooted; like it was going through its POST routine. 

I own my own cable modem, a Motorola Surfboard Cable Modem SB6141. Low and behold if you own this modem you can access it via a web interface and look at a whole bunch of stuff to help you troubleshoot it. Just navigate to <a href="http://192.168.100.1/index.htm" target="_blank">http://192.168.100.1/index.htm</a> and you will get the following screen (actually I think this works for a bunch of different modems as well).

<img alt='' class='alignnone size-full wp-image-1261 ' src='http://www.cjsommer.com/wp-content/uploads/2016/04/img_571ff0840569b.png' />

So the first place I went was to the signal levels page. After doing some research on the google I found a bunch of information regarding optimal signal levels for this modem. 

Downstream signal levels should be between +/-7dBmV, although I found some people that claimed they had no issue +/-15dBmV.

Downstream signal to noise ratio should be in the 32dB or higher range (higher is better).

Upstream signal levels should be between 44-50dBmV with a maximum of 55 dBmV. Beyond 55dBmV the cable modem appears to suffer stability issues resulting in rebooting of the cable modem.

So here were my initial cable modem signal levels. My signal to noise ratio is good, but my downstream levels are way too low and my upstream is way too high. I think at this point I know why I am rebooting all the time.
<img alt='' class='alignnone size-full wp-image-1257 ' src='http://www.cjsommer.com/wp-content/uploads/2016/04/img_5718b1701d938.png' />

So I called my cable provider and explained my situation. They gave me the boiler plate "your levels look good, please reboot all the things" response that I expected and I said "No, my signal levels are shit, please send a service rep." They reluctantly agreed to do so, but it wasn't going to be until the following week. That was not what I really wanted to hear so I decided to see what I could do to fix this myself. 

Well one way you can control signal levels is to take out any unnecessary splitters in the path to your cable modem. Each time you go through splitter you lose power (decibels).
<ul>
	<li>2-way - 3.5dB</li>
	<li>3-way balanced - 5.5dB</li>
	<li>3-way unbalanced - 3.5dB or 7dB</li>
	<li>4-way - 7dB </li>
	<li>8-way - 10.5dB</li>
</ul>
You get the picture. Splitters cause signal loss. Removing unnecessary splitters is one way to improve the signals to/from your cable modem.  

Here is a diagram of my RJ45 network before I did anything to it. 
<img alt='' class='alignnone size-full wp-image-1266 ' src='http://www.cjsommer.com/wp-content/uploads/2016/04/img_571ff6efa0ec8.png' />

You can see I had 2 open outputs so I had an opportunity to eliminate 2 splitters. I totally got rid of the initial 2-way and I replaced the 4-way with an unbalanced 3-way. Theoretically that should save me 7dB on the way to my cable modem. Here is a diagram of my RJ45 network after I removed the open circuits and removed the unnecessary splitters.

<img alt='' class='alignnone size-full wp-image-1267 ' src='http://www.cjsommer.com/wp-content/uploads/2016/04/img_571ff88fb2e50.png' />

Here are my signal levels after I took out unnecessary RJ-45 splitters. 

<img alt='' class='alignnone size-full wp-image-1256 ' src='http://www.cjsommer.com/wp-content/uploads/2016/04/img_5718b1248a33a.png' />

My downstream levels went up by about 7dB as I would have expected. My upstream levels went down within normal operating range and I haven't suffered any stability problems since. My internet had been suffering minor hiccups over the past 6 months or so and those have all gone away completely. Needless to say I called and cancelled my service call. Why mess with a good thing at this point?

Is this the only thing that can cause internet service problems? Absolutely not, but sometimes it pays to dig into these things yourself. If you like electronics like I do you'll probably have some fun and you will most definitely learn something along the way.  
