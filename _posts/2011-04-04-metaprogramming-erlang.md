---
ID: 138
post_title: Metaprogramming Erlang the Easy Way
author: Alex
post_excerpt: ""
layout: post
permalink: >
  https://alexbowe.com/metaprogramming-erlang/
published: true
post_date: 2011-04-04 17:49:46
---
<img src="https://alexbowe.com/wp-content/uploads/2011/04/a_hyper_cube.gif" alt="A Hyper Cube" width="398" height="339" class="aligncenter size-full wp-image-288" />

I&rsquo;ve recently taken <a href="http://www.erlang.org/">Erlang</a> back up<a id="fnref:0" class="footnote" title="see footnote" href="#fn:0">1</a>, and I wanted to use this blog post to talk about something cool I learned over the weekend.

I am implementing a data structure. Reimplementing actually, as it is the structure from my <a href="https://github.com/alexbowe/honours-thesis/downloads">thesis</a> - a succinct text index (I will post a blog on this soon).

Why am I reimplementing it in Erlang? The structure involves many bit-level operations, and I wanted to try out Erlang&rsquo;s <a href="http://www.erlang.org/doc/programming_examples/bit_syntax.html">primitive Binary type</a>, which seems to be allow efficient splitting and concatenation (which I require). Erlang&rsquo;s approach to concurrency will hopefully assist me to experiment with distributing the structure, too. As a bonus, the functional approach has lent itself well to this math-centric data structure, and my code is much, MUCH cleaner because of it (I love <a href="http://en.wikipedia.org/wiki/Fold_(higher-order_function)">reduce</a> and other <a href="http://en.wikipedia.org/wiki/Higher-order_function">higher order functions</a>).

One function I needed to implement was <a href="http://graphics.stanford.edu/~seander/bithacks.html#CountBitsSetTable">popcount</a> (the sum of set set bits in a bitvector). For example, if <code>b = 1010</code> then <code>pop(b)</code> is 2.

There are many methods listed on <a href="http://www.valuedlessons.com/2009/01/popcount-in-python-with-benchmarks.html">this blog</a>. One of them is the table method, which precomputes all popcounts for 16 bit integers (or any length you have space for):

<pre><code>POPCOUNT_TABLE16 = [0] * 2**16
for index in xrange(len(POPCOUNT_TABLE16)):
    POPCOUNT_TABLE16[index] = (index &amp; 1)
    + POPCOUNT_TABLE16[index &gt;&gt; 1]</code></pre>

I translated this to Erlang:

https://gist.github.com/901235

So now <code>gen_table(Bits)</code> will generate a tuple for me with all popcounts from 0 to $2^Bits$. However, we may gain performance<a id="fnref:1" class="footnote" title="see footnote" href="http://#fn:1">2</a> from knowing how big we want the table to be at compile time. If we know the amount of bits we want our popcount table to work for, we could type the table directly into the source. But that would impede our flexibility, and make our code ugly.

Enter <a href="https://github.com/esl/parse_trans"><code>ct_expand</code></a>. We can use <code>ct_expand:term( &lt;code to execute&gt; )</code> to run <code>gen_table()</code> for us at compile time! Now we only run <code>gen_table()</code> whenever we compile the module - after that the table is embedded in the binary.

First, check out ct_expand (<code>git clone https://github.com/esl/parse_trans.git</code>), and compile it with <code>make</code><a id="fnref:2" class="footnote" title="see footnote" href="#fn:2">3</a>. Then just move the <code>.beam</code> files into your project directory. Now we&rsquo;re ready for some metaprogramming :)

I created a new module for generating our table at compile time:

https://gist.github.com/901242

The reason we must be in a new module is because <code>ct_expand:term()</code> requires it&rsquo;s parameter to be something it will know about at compile time. This could be an inline fun (which I tried, but it wasn&rsquo;t pretty), or it can be something you compile before <code>ct_expand:term()</code> is executed (see line 3). Note on line 2 that we also need to compile <code>parse_transform</code> and <code>ct_expand</code> with our module.

Let&rsquo;s test it out:

<pre><code>1&gt; c(popcount).
{ok,popcount}
2&gt; popcount:popcount16(2#1010).
2
3&gt; popcount:popcount16(2#101011).
4</code></pre>

Nice! Thanks to <a href="http://ulf.wiger.net/weblog/">Ulf Wiger</a> for making that so easy :)

( Image stolen from&nbsp;http://improvisazn.wordpress.com/2010/02/22/meta/ )

<div class="footnotes">
<hr />
<ol>
<li id="fn:0">
My only other encounter with it was when I wrote an essay on the rationale of Erlang, which is something I will convert to blog format and post later if anyone is interested.<a class="reversefootnote" title="return to article" href="http://#fnref:0">&nbsp;↩</a>
</li>
<li id="fn:1">
The compiler may be able to apply further optimisations, so table access might be faster itself, but the main benefit comes from not having to generate the table while your code is running. Note that I haven&rsquo;t experimented with it. I will try to run some tests this week and update the blog post.<a class="reversefootnote" title="return to article" href="#fnref:1">&nbsp;↩</a>
</li>
<li id="fn:2">
If rebar gives you an error about crypto being undefined, you will need the <code>erlang-crypto</code> package. On Mac OS X: <code>sudo port install erlang +ssl</code>. (I had this issue)<a class="reversefootnote" title="return to article" href="http://#fnref:2">&nbsp;↩</a>
</li>
</ol></div>