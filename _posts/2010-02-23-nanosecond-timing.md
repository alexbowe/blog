---
ID: 206
post_title: Nanosecond Timing
author: Alex
post_excerpt: ""
layout: post
permalink: https://alexbowe.com/nanosecond-timing/
published: true
post_date: 2010-02-23 20:37:23
---
<img src="https://alexbowe.com/wp-content/uploads/2013/01/timer.jpg" alt="" width="620" height="360" class="alignnone size-full wp-image-207" />

<p>My Uni (RMIT) uses a mixture of Solaris, Mac OS X, Linux, and Windows computer labs. Our programs are nearly always tested on Solaris, though. Sometimes we are required to provide nanosecond timings in our experiments using the (real-time) POSIX function <code>gethrtime()</code>.</p>
<p>Depending on which lab I work in, or if I&rsquo;m working from home, I might need to comment out (or add compile guards) into my code to compile it correctly. This can make the code less readable, and can make its behavior (particularly output) on Solaris less obvious while testing on a foreign system.</p>
<p>Although the granularity, accuracy and any possible side effects (such as function overhead, or being affected by changing the system clock) may be different for each function, I find it helps to at least give a ball-park figure. <em>Experiments should still be done on Solaris</em>. If you want to use or modify my nanosecond function call wrapper, get it on <a href="http://github.com/alexbowe/nanotime_wrapper">GitHub</a>.</p>
<p><strong>Note</strong>: Not thoroughly tested on all target operating systems or compilers. It is only intended as a convenience.</p>