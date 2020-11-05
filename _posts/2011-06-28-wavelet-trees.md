---
ID: 106
post_title: 'Wavelet Trees &#8211; an Introduction'
author: Alex
post_excerpt: ""
layout: post
permalink: https://alexbowe.com/wavelet-trees/
published: true
post_date: 2011-06-28 11:35:50
---
<img src="http://alexbowe.s3.amazonaws.com/blog/overview.png" width="336" height="220" class="aligncenter" alt="A Wavelet Tree provides fast querying over a string using a hierarchy of compressed bitmaps."/>

Today I will talk about an elegant way of answering rank queries on sequences over <em>larger alphabets</em> - a structure called the Wavelet Tree. In <a href="https://alexbowe.com/yarrr-me-hearties">my last post</a> I introduced a data structure called RRR, which is used to quickly answer rank queries on <em>binary</em> sequences, and provide implicit compression.

A <em>Wavelet Tree</em> organises a string into a hierarchy of bit vectors. A rank query has time complexity is $&#92;mathcal{O}(&#92;log_2{A})$, where $A$ is the size of the alphabet. It was introduced by Grossi, Gupta and Vitter in their 2003 paper <em>High-order entropy-compressed text indexes</em> [4] (see the <em>Further Reading</em> section for more papers). It has since been featured in many papers [1, 2, 3, 5, 6].

If you store the bit vectors in RRR sequences, it may take less space than the original sequence. Alternatively, you could store the bit vectors in the rank indexes proposed by Sadakane and Okonohara [7]. It has a different approach to compression. I will talk about it another time ;) - fortunately, I will be studying under Sadakane-sensei at a later date (<em>update: now I'm doing my Ph.D. under him in Tokyo</em>).

In a different future post, I will show how Suffix Arrays can be used to find arbitrary patterns of length $P$, by issuing $2P$ rank queries.  If using a Wavelet Tree, this means a pattern search has $&#92;mathcal{O}(P &#92;log_2{A})$ time complexity, that is, the size of size of the 'haystack' doesn't matter, it instead depends on the size of the 'needle' and size of the alphabet.

<h2>Constructing a Wavelet Tree</h2>

A Wavelet Tree converts a string into a balanced binary-tree of bit vectors, where a $0$ replaces half of the symbols, and a $1$ replaces the other half. This creates <em>ambiguity</em>, but at each level this alphabet is filtered and re-encoded, so the ambiguity lessens, until there is no ambiguity at all.

The tree is defined recursively as follows:

<ol>
<li>Take the alphabet of the string, and encode the first half as $0$, the second half as $1$: $&#92;{ a, b, c, d &#92;}$ would become $&#92;{ 0, 0, 1, 1 &#92;}$;</li>
<li>Group each $0$-encoded symbol, $&#92;{ a, b &#92;}$, as a sub-tree;</li>
<li>Group each $1$-encoded symbol, $&#92;{ c, d &#92;}$, as a sub-tree;</li>
<li>Reapply this to each subtree recursively until there is only one or two symbols left (when a $0$ or $1$ can only mean one thing).</li>
</ol>

For the string <code>"Peter Piper picked a peck of pickled peppers"</code> (spaces and a string terminator have been represented as $&#92;_$ and $&#92;$$ respectively, due to convention in the literature) the Wavelet Tree would look like this:

<img src="http://alexbowe.s3.amazonaws.com/blog/binwt.png" width="440" height="261" class="aligncenter" alt="A Wavelet Tree for the string 'Peter Piper picked a peck of pickled peppers'."/>

<em>note: the strings aren't actually stored, but are shown here for convenience</em>

It has the alphabet $&#92;{ &#92;$, P, &#92;_, a, c, d, e, f, i, k, l, o, p, r, s, t &#92;}$, which would be mapped to $&#92;{ 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1 &#92;}$. So, for example, $&#92;$$ would map to $0$, and $r$ would map to $1$.

The left subtree is created by taking just the 0-encoded symbols $&#92;{ &#92;$, P, &#92;_, a, c, d, e, f &#92;}$ and then re-encoding them by dividing this <em>new</em> alphabet: $&#92;{ 0, 0, 0, 0, 1, 1, 1, 1 &#92;}$. Note that on the first level an $e$ would be encoded as a $0$, but now it is encoded as a $1$ (it becomes a $0$ again at a leaf node).

We can store the bit vectors in RRR structures for fast binary rank queries (which are needed, as described below), and compression :) In fact, since it is a balanced tree, we can concatenate each of the levels and store it as one single bit vector.

<h2>Querying a Wavelet Tree</h2>

Recall from <a href="https://alexbowe.com/yarrr-me-hearties">my last post</a> that a rank query is the count of $1$-bits up to a specified position. Rank queries over larger alphabets are analogous - instead of a $1$, it may be any other symbol:

<img src="http://alexbowe.s3.amazonaws.com/blog/rankalpha.png" width="315" height="129" class="aligncenter" alt="An example of the rank function."/>

After the tree is constructed, a rank query can be done with log $A$ ($A$ is alphabet size) <em>binary</em> rank queries on the bit vectors - $&#92;mathcal{O}(1)$ if you store them in RRR or another binary rank index. The encoding at each internal node may be ambiguous, but of course it isn't useless - we use the ambiguous encoding to guide us to the appropriate sub-tree, and keep doing so until we have our answer.

For example, if we wanted to know $rank(5, e)$, we use the following procedure which is illustrated below. We know that $e$ is encoded as $0$ at this level, so we take the <em>binary</em> rank query of $0$ at position $5$:

<img src="http://alexbowe.s3.amazonaws.com/blog/binwt-query_1.png" width="378" height="136" class="aligncenter" alt="The first step in a Wavelet Tree rank query."/>

Which is $4$, which we then use to indicate where to rank in the $0$-child: the $4^{th}$ bit (or the bit at position $3$, due to $0$-basing). We know to query the $0$-child, since that is what $e$ was encoded as at the parent level. We then repeat this recursively:

<img src="http://alexbowe.s3.amazonaws.com/blog/binwt-query.png" width="378" height="288" class="aligncenter" alt="A completed Wavelet Tree rank query."/>

At a leaf node we have our answer. I would love to explain why this works, but it is fun and rewarding to think about it yourself ;)

There are also ways to provide fast select queries, but once again I will leave that up to you to research. The curious among you might also be interested in the Huffman-Shaped Wavelet Tree described by M&auml;kinen and Navarro [5].

<h2>Using Your New Powers for Good</h2>

Feel free to implement this yourself, but if you want to get your hands dirty right away, all-around-clever-guy <a href="http://fclaude.recoded.cl">Francisco Claude</a> has made an implementation available in his <a href="http://libcds.recoded.cl">Compressed Data Structure Library (libcds)</a>. If you create something neat with it be sure to report back ;)

Update: <a href="http://siganakis.com/challenge-design-a-data-structure-thats-small">Terence Siganakis</a> wrote a blog post about Wavelet Trees that made it to the front page of Hacker News, encouraging an interesting discussion. The discussion is <a href="http://news.ycombinator.com/item?id=3650657">here</a>.

And if you read this far, consider following me on Twitter: <a href="http://www.twitter.com/alexbowe">@alexbowe</a>.

<h2>Further Reading</h2>

I didn't want to saturate this blog post with proofs and other details, since it was meant to be a light introduction. If you want to dive deeper into this beautiful structure, check out the following papers:

[1]  F. Claude and G. Navarro. Practical rank/select queries over arbitrary sequences. In Proceedings of the 15th International Symposium on String Processing and Information Retrieval (SPIRE), LNCS 5280, pages 176&ndash;187. Springer, 2008.

[2]  P. Ferragina, R. Giancarlo, and G. Manzini. The myriad virtues of wavelet trees. Information and Computation, 207(8):849&ndash;866, 2009.

[3]  P. Ferragina, G. Manzini, V. M ̈akinen, and G. Navarro. Compressed representations of sequences and full-text indexes. ACM Transactions on Algorithms, 3(2):20, 2007.

[4]  R. Grossi, A. Gupta, and J. Vitter. High-order entropy-compressed text indexes. In Proceedings of the 14th annual ACM-SIAM symposium on Dis- crete algorithms, pages 841&ndash;850. Society for Industrial and Applied Mathematics, 2003.

[5]  V. M&auml;kinen and G. Navarro. Succinct suffix arrays based on run-length encoding. Nordic Journal of Computing, 12(1):40&ndash;66, 2005.

[6]  V. M&auml;kinen and G. Navarro. Implicit compression boosting with applications to self-indexing. In Proceedings of the 14th International Symposium on String Processing and Information Retrieval (SPIRE), LNCS 4726, pages 214&ndash;226. Springer, 2007.

[7]  D. Okanohara and K. Sadakane. Practical entropy-compressed rank/select dictionary. Arxiv Computing Research Repository, abs/cs/0610001, 2006.