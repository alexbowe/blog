---
ID: 151
post_title: Regularly Divisble
author: Alex
post_excerpt: ""
layout: post
permalink: https://alexbowe.com/regularly-divisble/
published: true
post_date: 2010-11-24 07:26:37
---
<img src="https://alexbowe.com/wp-content/uploads/2010/11/div3.png" alt="div3" width="373" height="96" class="aligncenter size-full wp-image-311" />
<p><strong>Update</strong>: read the comments at  <a href="http://news.ycombinator.com/item?id=1937062">Hacker News</a> to see some succinct approaches to this, as discussed by <em>gjm11</em>,  <em>qntm</em> and <em>patio11</em>. Thanks to <em>Robin</em> for providing <a href="http://s3.boskent.com/divisibility-regex/divisibility-regex.html">this  demonstration</a> that can find a regex for testing divisibility of any number, in any base (he  also made the code available, nice).</p>
<p>Earlier this year, at the advice (once more) of <a href="http://www.amazon.com/gp/product/1934356344/alexbowecom-20">Chad Fowler</a>, I took to the idea of practicing programming every day. Perhaps this appealed to me because it echoed the rituals of my better musician friends, and allowed me to draw parallels between programming and my fading dream of becoming a famous rockstar.</p>
<p>Possibly because of my failed interview at Google (hey, I wouldn't have hired  the back-then me either, so no hard feelings!), I was also interested in  job-interview styled problems [1]. <em>Not</em> <a href="http://niki.code-karma.com/2010/08/fizzbuzz/">FizzBuzz</a> though, more like  the computer science 'riddles' found on <a href="http://wuriddles.com/cs.shtml">this page</a> [2].</p>
<p>At the time I was teaching Computing Theory [3], 80% of which was <a href="http://en.wikipedia.org/wiki/Formal_language">formal  languages</a>: <a href="http://en.wikipedia.org/wiki/Regular_expression">regular expressions</a>, <a href="http://en.wikipedia.org/wiki/Context-free_grammar">context free</a> and  <a href="http://en.wikipedia.org/wiki/Context-sensitive_grammar">context sensitive grammars</a>, <a href="http://en.wikipedia.org/wiki/Turing_machine">Turing machines</a> and <a href="http://en.wikipedia.org/wiki/Automata_theory">other  automata</a>, and their locations in the <a href="http://en.wikipedia.org/wiki/Chomsky_hierarchy">Chomsky Hierarchy</a>.  So, this problem appealed to me:</p>
<blockquote>
<p>Construct a finite state machine (or equivalently, write a regular  expression) which accepts all strings over the alphabet {0,1} which are  divisible by 3 when interpreted in binary.</p>
</blockquote>
<p>It is pretty interesting that languages can be defined to communicate patterns  in binary sequences that are divisible by 3. Let's solve it in more detail than  necessary :)...</p>
<p><!--more--></p>
<p>We will be developing this regex incrementally. I will update the regex at each stage. Since it is easier to construct this regex using a finite state machine  (which is how I worked this out the first time), I'll also include a few  diagrams along the way. You can test your regexes using the Ruby code  <a href="https://gist.github.com/711644">here</a>.</p>
<p>To get an idea for the pattern, I started by listing out a few multiples of 3  and their binary representations. You can use the following Ruby code if you  want.</p>

https://gist.github.com/709642

<pre><code>&gt;&gt; 30.times {|n| puts "#{n}: #{n.to_bin_s}" if n%3 == 0}
0: 0
3: 11
6: 110
9: 1001
12: 1100
15: 1111
18: 10010
21: 10101
24: 11000
27: 11011
</code></pre>
<p>Observe that the binary representations for 3, 6, 12 and 24 have something in  common: they have the same sequence, with a little bit of zero-padding to the  right. <em>If <code>X</code> is a binary string divisible by <code>3</code>, then the binary string <code>X0</code> is divisible by <code>3</code> as well</em>. This makes sense, as adding zeros to the right is  equivalent to multiplying by <code>2</code>, and we know that <code>n * 3 * 2</code> is divisible by  <code>3</code> for any integer <code>n</code>. This gives us:</p>
<pre><code>r = A0*
</code></pre>
<p><em>A will represent the leftover regex at each stage</em>.</p>
<p>This can trivially be applied to left zero-padding as well, because a binary  string <code>X</code> is numerically equivalent to the binary string <code>0X</code> (issues of  <a href="http://en.wikipedia.org/wiki/Endianness">endianness</a> aside):</p>
<pre><code>r = 0*A0*
</code></pre>
<p>Since we already covered even multiples, to work out <code>A</code> we just need to look at odd multiples of <code>3</code>. Here are a few:</p>
<pre><code>&gt;&gt; 50.times {|n| puts "#{n}: #{n.to_bin_s}" if n%3 == 0 and n%2 != 0}
3: 11
9: 1001
15: 1111
21: 10101
27: 11011
33: 100001
39: 100111
45: 101101
</code></pre>
<p>You might have noticed that <code>1111</code> (15) is just <code>11</code> (3) concatenated with itself. That's the same as saying <code>15 = 3 * 2 * 2 + 3</code>. If you add two multiples of  three, you will of course get another multiple of 3 (it's the same for any  multiplier), and concatenating just involves doubling the first number a few  times pre-addition. Hence, you will always get another multiple of 3 by  concatenating two.</p>
<pre><code>r = (0*A0*)+
</code></pre>
<p>From observation, here are the above numbers that are <em>not</em> concatenations:</p>
<pre><code>3: 11
9: 1001
21: 10101
33: 100001
45: 101101
</code></pre>
<p>Take note that it always starts and ends with a <code>1</code>. It should definitely  end with a <code>1</code> to be odd, and start with a different <code>1</code> to be greater than  <code>1</code>. Our updated regex becomes:</p>
<pre><code>r = (0*1A10*)+
</code></pre>
<img src="https://alexbowe.com/wp-content/uploads/2010/11/autom1.png" alt="autom1" width="362" height="203" class="aligncenter size-full wp-image-307" /><p>It also appears that we can optionally insert an even number of zeros between  the end <code>1</code>s. A proof of this is attached at the end, if you're into that sort of thing.</p>
<pre><code>r = (0*1(0A0)*10*)+
</code></pre>
<img src="https://alexbowe.com/wp-content/uploads/2010/11/autom2.png" alt="autom2" width="362" height="244" class="aligncenter size-full wp-image-309" />
<p>So what about those <code>1</code>s in the middle then? It appears that we're allowed to  have 0, 1 or 2... maybe more consecutive 1s? Maybe we can say our regex is <code>r = (0*1(01*0)*10*)+</code>. I'll <a href="https://gist.github.com/711644">test</a> the regex for the first 5000 integers:</p>
<pre><code>False Negatives:
  0
</code></pre>
<p>Oops! I forgot about <code>0</code> being evenly divisible by <code>3</code> exactly <code>0</code> times. Our  regex should account for this:</p>
<pre><code>r = 0|(0*1(01*0)*10*)+
</code></pre>
<img src="https://alexbowe.com/wp-content/uploads/2010/11/autom3.png" alt="autom3" width="362" height="319" class="aligncenter size-full wp-image-310" />
<p>Running the test again we get this output:</p>
<pre><code>Pass =]
</code></pre>
<p>So there you have it, the regex works for the first 5000 integers. I'll leave a  proof of this for all multiples of 3 as an exercise to the reader ;)</p>
<hr />
<p>[1] I'm not sure I want to work for someone else anymore - I'd rather chase my  own stupid dreams. Probably not the rockstar one though. I'll write about these later.</p>
<p>[2] Other good sources of practice questions are <a href="http://www.amazon.com/gp/product/0201657880/alexbowecom-20">Programming Pearls</a>, <a href="http://projecteuler.net/">Project Euler</a>, or you could take a paper from <a href="http://scholar.google.com.au/scholar?as_q=&amp;num=10&amp;btnG=Search+Scholar&amp;as_epq=&amp;as_oq=&amp;as_eq=&amp;as_occt=any&amp;as_sauthors=&amp;as_publication=acm+transactions+on+algorithms&amp;as_ylo=&amp;as_yhi=&amp;as_sdt=1.&amp;as_sdtp=on&amp;as_sdts=5&amp;hl=en">ACM Transactions on Algorithms</a><br /> (for example) and implement the algorithm/data structure. While you're at it,  why not learn <a href="http://www.erlang.org/">Erlang</a> (or any other programming language guaranteed to  make you more appealing to the opposite sex) and implement it in that?</p>
<p>[3] Recently, a past student of mine told me that they have never found  Computing Theory to be of any use. I said "What about regular expressions?" and  they shook their head. This was a smart student too, but I find that regular  expressions are so damn useful <a href="http://m68k.net/2010/09/01/stop-searching-for-regexes.html">(perhaps a hammer I swing too often)</a>.  Ironically, Computing <em>Theory</em> was the most practically useful subject in my  (watered down) degree. In the near future I intend on climbing on my high-horse  and writing a blog post about my ideal CS degree.</p>
<hr />
<h2>Appendix</h2>
<p>Here is a proof that shows we can optionally insert an even number of zeros  between two <code>1</code>s to get an odd multiple of 3. This is equivalent to saying that  our left-most <code>1</code> has to be in an odd position index (if 0 is the rightmost...).</p>
<pre><code>RTP: 2^(2i + 1) + 1 = 3m, for m odd integer and i positive integer or 0

Let i = 0, then
LHS = 2^1 + 1
    = 2 + 1
    = 3 * 1
    = 3m (1 is an odd integer)
    = RHS
    =&gt; it holds for i = 0

Let i = k for any integer k, then assume
2^(2k + 1) + 1 = 3m

Let i = k + 1, then
LHS = 2^[2(k + 1) + 1] + 1
    = 2^(2k + 1 + 2) + 1
    = 4 * 2^(2k + 1) + 1
    = 3 * 2^(2k + 1) + 3m'
    = 3[ 2^(2k + 1) + m']
    = 3[ 2^(2k + 1) + 1 + (m' - 1)]
    = 3[3m' + (m' - 1)]
    = 3(4m' - 1)
    = 3m (an even number minus 1 will be odd)
    = RHS
QED
</code></pre>
<p>Yes, 2^n + 1 always yields an odd multiple of 3, for n odd.  I should really set  up a math plugin...</p>