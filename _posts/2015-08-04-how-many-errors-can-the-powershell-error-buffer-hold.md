---
layout: post
title: How many errors can the PowerShell error buffer hold?
date: 2015-08-04 09:00
author: cjsommer@gmail.com
comments: true
categories: [PowerShell]
---
<hr>
During my presentation at SQL Saturday Albany I was asked a question about the error buffer that I didn't know the answer to. The question was, "How many errors can the error buffer hold?" and I really had no idea. I had a few minutes after the presentation and decided to figure it out on the spot, but I also thought it would make a nice blog post. 

PowerShell stores errors that it encounters in a circular error buffer variable named $Error. $Error also contains a count property to count the number of errors in the buffer. To figure out how many errors we could hold was pretty straight forward. All I had to do was create a script that generated an error and see how high $error.count got. At this point I didn't even know if there was a maximum or not.

<pre class="lang:ps decode:true " title="Find the maximum number of errors " >
$ErrorCount = 0
 do {
    $LastErrorCount = $ErrorCount
    Get-ChildItem "c:\bogusfile.txt" -ErrorAction SilentlyContinue
    $ErrorCount = $Error.Count
} while ($LastErrorCount -ne $ErrorCount)

"There are $ErrorCount errors in the error buffer."
</pre> 

Low and behold there is a maximum number of errors that the error buffer can hold. That number is 256, which just so happens to be the default.
<a href="/img/2015/07/256errors.png"><img src="/img/2015/07/256errors.png" alt="256errors" width="489" height="56" class="alignnone size-full wp-image-878" /></a>

After finding a maximum of 256, I also wondered if that was something I had control over. So I dug a little deeper, and sure enough there is a way to control the maximum size of the error buffer. There is a $MaximumErrorCount variable made exactly for that purpose (nice, the name even makes sense). Just as a quick test I set $MaximumErrorCount to 512, and reran my test.
 
<pre class="lang:ps decode:true " title="MaximumErrorCount = 512" >
$MaximumErrorCount = 512
$ErrorCount = 0
 do {
    $LastErrorCount = $ErrorCount
    Get-ChildItem "c:\bogusfile.txt" -ErrorAction SilentlyContinue
    $ErrorCount = $Error.Count
} while ($LastErrorCount -ne $ErrorCount)
''
"There are $ErrorCount errors in the error buffer."
</pre> 

Sure enough, I now had 512 errors in my error buffer.

<a href="/img/2015/07/512errors.png"><img src="/img/2015/07/512errors.png" alt="512errors" width="424" height="49" class="alignnone size-full wp-image-879" /></a>

I had never really thought about this before and I'm not sure I'd ever need to handle more than the default number of 256 errors, but this is another nice to know fact. I've been working with PowerShell for quite some time and this is just another example that there's always something left to learn. 

Happy scripting, everyone!

