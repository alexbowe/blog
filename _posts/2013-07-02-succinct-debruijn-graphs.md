---
ID: 386
post_title: Succinct de Bruijn Graphs
author: Alex
post_excerpt: ""
layout: post
permalink: >
  https://alexbowe.com/succinct-debruijn-graphs/
published: true
post_date: 2013-07-02 21:07:35
---
<p>This post will give a brief explanation of a Succinct implementation for storing <a href="http://en.wikipedia.org/wiki/De_Bruijn_graph">de Bruijn graphs</a>, which is recent (and continuing) work I have been doing with Sadakane.</p>

<p>Using our new structure, we have squeezed a graph for a human genome (which took around 300 GB of memory if using previous representations) down into 2.5 GB.
In addition, the construction method allows much of the work to be done on disk. Your computer might not have 300 GB of RAM, but you might have 2.5 GB of RAM and a hard disk.</p>

<p>I have given a talk about this a few times, so I&#8217;ve been itching to write it up as a blog post (if only to shamelessly plug my blog at conferences).
However, since it is a lowly blog post, I won&#8217;t give attention to the gory details, nor provide any experimental results. But
this post is merely to <em>communicate the approach</em>. Feel free to check out our <a href="https://www.dropbox.com/s/cxxuqjs663fdth7/succinctdebruijn2012.pdf">conference paper</a>, and stay tuned for the journal paper.</p>

<p>In this blog post, I will first give an introduction to <a href="#debruijn-graphs">de Bruijn graphs</a> and <a href="#dna-assembly">how they are used in DNA assembly</a>. Then I will
briefly explain <a href="#previous">some previous implementations</a> before reaching the main topic of this post: <a href="#our-repr">our new succinct representation</a>.
The explanation first explains some preliminaries, such as how we were <a href="#bwt-inspiration">inspired by the Burrows Wheeler Transform</a>, and what <a href="#ranksel">rank and select</a> are,
which are required to understand the <a href="#construction">construction method</a>, and <a href="#interface">traversal interface</a> respectively.</p>

<p>For those following along at home, I have implemented <a href="https://github.com/alexbowe/debby/blob/0.1.1/debby.py">a demo version in Python</a>. It doesn&#8217;t use efficient implementations of rank and select, nor provide any compression -
it is merely meant to demonstrate the key ideas with (hopefully) readable high level code. An optimised version will be made available at some point.</p>

<p>Okay, here we go&#8230;</p>

<br/><br/>
<h2 id="debruijn-graphs">De Bruijn Graphs</h2>

<p>De Bruijn graphs are a beautifully simple, yet useful combinatoric object which I challenge you not to lose sleep over.</p>

<p>Since their discovery around 1946, they have been used in a variety of applications, such as encryption, psychology,
chess and even card tricks. Quite recently they have become a popular data structure for DNA assembly of short read data.</p>

<p>They are defined as a directed graph, where each node $u$ represents a fixed-length string (say, of length $k$,
and an edge exists from $u$ to $v$ iff they overlap by $k&#8211;1$ symbols. That is, $u[2..k] = v[1..k&#8211;1]$.</p>

<p>Let&#8217;s make this concrete with a diagram:</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/23debruijn.png width=240px />
</figure>
</center>

<p>Doesn&#8217;t this just <em>feel good</em> to look at? Now tilt your head and look at it this way: it is essentially a <a href="https://en.wikipedia.org/wiki/Finite-state_machine">Finite State Machine</a>
with additional bounded memory of the last $k$ visited states, if you added a few more states to allow for incomplete input at the start.
For example, if X represents blank input, from a starting state XXX we might have the transition chain: XXX -&gt; XX1 -&gt; X10 -&gt; 101 (which is already a node in the graph).
Put this way, it is easy to see that the $k$ previous edges define the current node. This perspective will make things easier to understand later, I promise.</p>

<p>Still with me? Good (: Then let&#8217;s also consider that if <em>one node</em> is defined by a $k$-length string,
then a <em>pair of nodes</em> (i.e. an edge) can be identified by a $k+1$ length string, since they overlap and
differ by 1 symbol. This will also be important later.</p>

<p>And of course, this can be extended to larger alphabet sizes than binary (say, 4&#8230;).</p>

<br/><br/>
<h2 id="dna-assembly">DNA Assembly</h2>

<p>First suggested in 2001 by Pevzner et al.<a href="#fn:1" id="fnref:1" title="see footnote" class="footnote">[1]</a>, we can use de Bruijn graphs to represent a network of overlapping
<em>short read data</em>.</p>

<p>The long and short of it (heh heh) is that a DNA molecule is currently too difficult to
sequence (that is, read it into a computer) in its entirety. Special methods must be used
so we can sequence parts of the molecule, and hand off the putting-back-together process
(assembly) to an algorithm.</p>

<p>One current popular sequencing method is <em>shotgun sequencing</em>, which clones the genome a
bunch of times, then randomly breaks each clone into short segments. If we can sequence
the short segments, then the fact that we randomly cut up the clones should lead us to have overlapping reads.
Of course it is a bit more complicated than this in reality, but this is the essence of it.</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/shotgun.png width=500px />
</figure>
</center>

<p>We then move a sliding window over each read, outputing overlapping $k$-mers (the $k$-length strings), which we use to create our de Bruijn graph.</p>

<p>About now mathematicians among you might raise your hand to tell me &#8220;that may not technically yield a de Bruijn graph&#8221;. That&#8217;s correct -
in Bioinformatics the term &#8220;de Bruijn graph&#8221; is overloaded to mean a subgraph. Even though genomes are long strings, most genomes won&#8217;t
have <em>every single k-mer</em> present, and there is usually repeated regions. This means our data will be sparse.</p>

<p>Consider the following contrived example. Take the sequence <code>TACGACGTCGACT</code>. If we set <span class="math">\(k=3\)</span>, our k-mers will be <code>TAC</code>, <code>ACG</code>, <code>CGA</code>, and so on.
We would end up with this de Bruijn graph:</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/graph.png />
</figure>
</center>

<p>After we construct the graph from the reads, assembly becomes finding the &#8220;best&#8221; contiguous regions.
The jury is still out on what the best method is (or what &#8220;best&#8221; even means); the point of this post isn&#8217;t about assembly, but our implementation of the
data structure and how it provides all required navigation options to implement any traversal method. I recommend reading
<a href="http://www.cs.ucdavis.edu/~gusfield/cs225w12/deBruijn.pdf">this primer from Nature</a> if you want to get deeper into this,.</p>

<p>Even though this is only a de Bruijn <em>subgraph</em>, these things still grow pretty big. It is worthwhile considering how to handle this scalability issue,
if only to reduce hardware requirements of sequencing (thus proliferating personal genomics), and potentially improve traversal speed (due to better memory locality).
Increased efficiency might also enable richer multiple-genomic analysis.</p>

<br/><br/>
<h2 id="previous">Previous Representations</h2>

<p>One of the first approaches to this was to scale &#8220;horizontally&#8221;. Simpson et al.<a href="#fn:2" id="fnref:2" title="see footnote" class="footnote">[2]</a> introduced ABySS in 2009. The <a href="ftp://ftp.ddbj.nig.ac.jp/ddbj_database/dra/fastq/SRA010/SRA010896/SRX016231/">graph for reads from a human genome (HapMap: NA18507)</a>,
which used a distributed hash table, reached 336 GB.</p>

<p>In 2011, Conway and Bromage<a href="#fn:3" id="fnref:3" title="see footnote" class="footnote">[3]</a> instead approached this problem from a &#8220;vertical&#8221; scaling perspective
(that is, scaling to make better use of a single system’s resources), by using a sparse bitvector (by Okanohara and Sadakane<a href="#fn:4" id="fnref:4" title="see footnote" class="footnote">[4]</a>) to represent the $(k + 1)$-mers (the edges),
and used <a href="#ranksel">rank and select</a> (to be described shortly) to traverse it. As a result, their representation took 32 GB for the same data set.</p>

<p>Minia, by Cikhi and Rizk (2012)<a href="#fn:5" id="fnref:5" title="see footnote" class="footnote">[5]</a>, proposed yet another approach by using a <a href="http://en.wikipedia.org/wiki/Bloom_filter">bloom filter</a> (with additional structure to avoid false positive edges that would
affect the assembly). They traverse by generating possible edges and testing for it in the bloom filter. Using this approach, the graph was reduced to 5.7 GB.</p>

<br/><br/>
<h2 id="our-repr">Our Succinct Representation</h2>

<p>As stated, we were able to represent the same graph in 2.5 GB (after some further compression techniques, which I will save for a future post).</p>

<p>The key insight is that the edges define overlapping node labels. This is similar to that of Conway and Bromage, although they have
some redundancy, since some <em>nodes</em> are represented more times than necessary.</p>

<p>We further exploit the <a href="https://en.wikipedia.org/wiki/Mutual_information">mutual information</a> of edges by taking inspiration from the Burrows Wheeler Transform<a href="#fn:6" id="fnref:6" title="see footnote" class="footnote">[6]</a>.</p>

<br/><br/>
<h3 id="bwt-inspiration">Inspiration from the Burrows Wheeler Transform</h3>

<p>The Burrows Wheeler Transform<a href="#fn:6" title="see footnote" class="footnote">[6]</a> is a reversible string permutation that can be searched directly and has the admirable quality of
having long strings of repeated characters (great for compression). The easiest way to calculate the BWT of a string is to
sort each symbol by their prefixes in colex order (that is, alphabetic order of the reverse of the string, not reverse alphabetic!)
More information can be found on <a href="http://en.wikipedia.org/wiki/Burrows%E2%80%93Wheeler_transform">Wikipedia</a> and this <a href="http://marknelson.us/1996/09/01/bwt/">Dr. Dobbs</a>
article.</p>

<p>The XBW is a generalisation of the BWT that applies to rooted, labeled trees<a href="#fn:7" id="fnref:7" title="see footnote" class="footnote">[7]</a>. The idea is that instead of taking all suffixes,
we sort all paths from the root to each node, and support tree navigation (since it isn&#8217;t a linearly shaped string) with
auxiliary bit vectors indicating which edges are leaves, and which are the last edges (of their siblings) of internal nodes.</p>

<p>I won&#8217;t go into detail, but in the next section you should be able to see glimpses of these two ideas.</p>


<br/><br/>
<h3 id="construction">Construction</h3>

<p>The simplest construction method<a href="#fn:8" id="fnref:8" title="see footnote" class="footnote">[8]</a> is to take every &lt;node, edge&gt; pair and sort them based on the reverse of the node label (colex order), removing
duplicates<a href="#fn:9" id="fnref:9" title="see footnote" class="footnote">[9]</a>. We also padding to ensure every node has an incoming and an outgoing edge. This maintains the fact that a node is defined by its previous k edges.
An example can be seen below:</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/graph-padded.png />
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/arrays-1.png />
</figure>
</center>

<p>You may have spotted that we have flagged some edges with a minus symbol. This is to disambiguate identically labelled incoming edges - edges that exit separate nodes,
but have the same symbol, and thus enter the same node. In the example below, the nodes <code>ACG</code> and <code>CGA</code> both have two incoming edges.</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/edges.png />
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/array-edges-flags.png />
</figure>
</center>

<p>Notice that each outgoing edge is stored contiguously? We include a bit vector to represent whether an edge is the <em>last</em> edge exiting a node.
This means that each node will have a sequence of zero-or-more 0-bits, followed by a single 1-bit.</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/last-edges.png />
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/last-array.png />
</figure>
</center>

<p>Since a 1 in the $L$ vector identifies a unique node, we can use this vector (and <a href="#ranksel">select</a>, explained shortly) to index nodes, whereas standard array indexing
points to edges.</p>

<p>Finally, instead of storing the node labels we just need to store the final column of the node labels. Since the node labels are sorted, it is equivalent to store an array of first
positions:</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/f-array.png />
</figure>
</center>

<p>In total we have a bitvector $L$, an array of flagged edge labels $W$, and a position array $F$ of size $\sigma$ (the alphabet size). Respectively, these take $m$ bits,
$m \log{2*\sigma} = 3 m$ bits (for DNA), and $\sigma \log{m} = o(m)$ bits<a href="#fn:10" id="fnref:10" title="see footnote" class="footnote">[10]</a>, given $m$ edges - a bit over 4 bits per edge.
Using appropriate structures (not detailed) we can compress this further, to around 3 bits per edge.</p>


<br/><br/>
<h3 id="ranksel">Rank and Select</h3>

<p>Rank and select are the bread and butter of succinct data structures, because so many operations can be implemented using them alone.
$rank_c(i)$ returns the number of occurences of symbol $c$ on the closed range $[0, i]$, whereas $select_c(i)$ returns the position of the $i^{th}$
occurence of symbol $c$. </p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/ranksel.png width=600px/>
</figure>
</center>

<p>They are <em>kind of</em> like inverse functions, although $rank()$ is not <a href="http://en.wikipedia.org/wiki/Injective_function">injective</a>,
so cannot have a true inverse.
For this reason, if you want to find the <em>position</em> of the left-nearest $c$, you would have to use $select()$ in orchestra with $rank()$ (a common pattern).</p>

<p>Speaking of patterns of use, it may also help to keep this in mind:
rank is for counting, and select is for searching, or can be thought of
as an indirect addressing technique (such as addressing a node using the $L$ array).
Two rank queries can count over a range, whereas two select queries can find a range.
A rank and a select query can find a range where either a start point or end point are fixed.</p>

<p>Rank, select and standard array access can all be done in $\mathcal{O}(1)$ time when $\sigma = polylog(N)$<a href="#fn:11" id="fnref:11" title="see footnote" class="footnote">[11]</a>,
if represent a bitvector using the structure described by Raman, Raman and Rao in 2007<a href="#fn:12" id="fnref:12" title="see footnote" class="footnote">[12]</a> (which I explained in <a href="https://alexbowe.com/rrr">an earlier blog post</a>), and for larger alphabets
use the index described by Ferragina, Manzini, Makinen, and Navarro in 2006<a href="#fn:13" id="fnref:13" title="see footnote" class="footnote">[13]</a>. In our implementation, we use modified versions to get it down to 3 bits per edge.</p>


<br/><br/>
<h3 id="interface">Interface Overview</h3>

<p>While it might not be obvious, these three arrays provides support for a full suite of navigation operations.
An overview is given in these tables, which link to the implementation details that follow.</p>

<p>First, to navigate each of the edges, we define two internal functions (using <a href="#ranksel">rank and select calls</a>):</p>

<table>
<colgroup>
<col style="text-align:left;"/>
<col style="text-align:left;"/>
<col style="text-align:left;"/>
</colgroup>

<thead>
<tr>
	<th style="text-align:left;">Operation</th>
	<th style="text-align:left;">Description</th>
	<th style="text-align:left;">Complexity</th>
</tr>
</thead>

<tbody>
<tr>
	<td style="text-align:left;"><a href="#forward"><span class="math">\(forward(i)\)</span></a></td>
	<td style="text-align:left;">Return index of the <em>last edge</em> of the <em>node pointed to</em> by edge <span class="math">\(i\)</span>.</td>
	<td style="text-align:left;"><span class="math">$\mathcal{O}(1)$</span></td>
</tr>
<tr>
	<td style="text-align:left;"><a href="#backward"><span class="math">\(backward(i)\)</span></a></td>
	<td style="text-align:left;">Return index of the <em>first edge</em> that <em>points to the node</em> that the edge at <span class="math">\(i\)</span> <em>exits</em>.</td>
	<td style="text-align:left;"><span class="math">\(\mathcal{O}(1)\)</span></td>
</tr>
</tbody>
</table>
<p>Using these two functions, we can implement the less confusing public interface below, which operate on node indexes:</p>

<table>
<colgroup>
<col style="text-align:left;"/>
<col style="text-align:left;"/>
<col style="text-align:left;"/>
</colgroup>

<thead>
<tr>
	<th style="text-align:left;">Operation</th>
	<th style="text-align:left;">Description</th>
	<th style="text-align:left;">Complexity</th>
</tr>
</thead>

<tbody>
<tr>
	<td style="text-align:left;"><a href="#outdegree"><span class="math">\(outdegree(v)\)</span></a></td>
	<td style="text-align:left;">Return number of outgoing edges from node <span class="math">\(v\)</span>.</td>
	<td style="text-align:left;"><span class="math">\(\mathcal{O}(1)\)</span></td>
</tr>
<tr>
	<td style="text-align:left;"><a href="#outgoing"><span class="math">\(outgoing(v,c)\)</span></a></td>
	<td style="text-align:left;">From node <span class="math">\(v\)</span>, follow the edge labeled by symbol <span class="math">\(c\)</span>.</td>
	<td style="text-align:left;"><span class="math">\(\mathcal{O}(1)\)</span></td>
</tr>
<tr>
	<td style="text-align:left;"><a href="#label"><span class="math">\(label(v)\)</span></a></td>
	<td style="text-align:left;">Return (string) label of node <span class="math">\(v\)</span>.</td>
	<td style="text-align:left;"><span class="math">\(\mathcal{O}(k)\)</span></td>
</tr>
<tr>
	<td style="text-align:left;"><a href="#indegree"><span class="math">\(indegree(v)\)</span></a></td>
	<td style="text-align:left;">Return number of incoming edges to node <span class="math">\(v\)</span>.</td>
	<td style="text-align:left;"><span class="math">\(\mathcal{O}(1)\)</span></td>
</tr>
<tr>
	<td style="text-align:left;"><a href="#incoming"><span class="math">\(incoming(v,c)\)</span></a></td>
	<td style="text-align:left;">Return predecessor node starting with symbol <span class="math">\(c\)</span>, that has an edge to node <span class="math">\(v\)</span>.</td>
	<td style="text-align:left;"><span class="math">\(\mathcal{O}(k \log \sigma) \)</span></td>
</tr>
</tbody>
</table>
<p>The details of the above functions are given in the following sections.</p>


<br/><br/>
<h3 id="forward">Forward</h3>

<p>In order to support the public interface, we create for ourselves a simpler way to work with edges: the complementing forward and backward functions.</p>

<p>Recall that all node labels are defined by predecessor edges, then we have represented each edge in two different places: the <span class="math">\(F\)</span> array (which is equivalent to
the last column of the &#8220;Node&#8221; array), and the edge array <span class="math">\(W\)</span>. It follows that, since it is sorted, the node labels maintain the same <em>relative order</em> as the
edge labels. This can be seen in the following figure:</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/fwdsetup.png />
</figure>
</center>

<p>Note that the number of <code>C</code>s in the last column of Node is different from the number of <code>C</code>s in <span class="math">\(W\)</span>, because the first <code>C</code> in <span class="math">\(W\)</span> points to two edges.
For this reason, we ignore the first edge from node <code>GAC</code> (although it doesn&#8217;t affect the relative order). In fact, we ignore any edge that doesnt have L[i] == 1.</p>

<p>Then, following an edge is simply finding the corresponding relatively positioned node! All it takes is some creative counting, using <a href="#ranksel">rank and select</a>, as pictured
below:</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/fwd.png />
</figure>
</center>

<p>First we access W[i] to find the edge label, then calculate <span class="math">\(rank_C\)</span> up to row to gives us the relative ordering of our <em>edges</em> with this label. Let&#8217;s call this relative index
<span class="math">\(r\)</span>. In our example we are following the 2nd C-labeled edge.</p>

<p>To find the 2nd occurence of C in F, first we need to know where the first occurence is. We can use F to find that. Then we can select to the 2nd one, using the last array.
Because the last array is binary only, this requires us to count how many 1s there are before the run of Cs (using rank), then adding 2 to land us at the 2nd C.</p>

<p>The W access, rank over W, rank and select over L, accessing F, and the addition are all done in O(1) time, so forward also takes O(1) time.</p>


<br/><br/>
<h3 id="backward">Backward</h3>

<p>Backward is very similar to forward, but calculated in a different order: we find the relative index of the node label first, and use that to find the corresponding edge (which may not be the
only edge to point to this node, but we define it to point to the first one, that is one that isn&#8217;t flagged with a minus).</p>

<p>We can find our relative index of the node label by issuing two rank queries instead, and using select on W to find the first incoming edge.</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/bwd.png />
</figure>
</center>

<p>For similar reasons to Forward this is O(1).</p>

<br/><br/>
<h3 id="outdegree">Outdegree</h3>

<p>This is an easy one. This function accepts a <strong>node</strong> (not edge!) index <code>v</code>, and returns the number of outgoing edges from that node. Why did I say this was easy?
Well, remember that all our outgoing edges from the same node are contiguous in our data structure. See the diagrams below, paying attention to the node <code>ACG</code>.</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/outdegree-graph.png />
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/outdegree-arrays.png />
</figure>
</center>

<p>By definition (since nodes are unique and sorted), this will always be the case. We also defined our so-called &#8220;last&#8221; vector $L$
to have a 0 for every edge from a given node, <em>except the last</em> edge. All we need to do is count how many 0s there
are, then add 1. Put another way, we need to measure the distance between 1s (since the previous 1 will belong to the previous node). Since we know the node index, we can use select for that!</p>

<p>In the above example, we are querying the outdegree of node 6 (the 7th node due to zero-basing). First we select to find the position of the 7th 1, which gives us the last
edge of that node. Then we simply subtract the position of the previous node (node 5, the 6th node): <span class="math">\(select(7) - select(6) = 8 - 6 = 2\)</span>. Boom.</p>

<p>Select queries can be answered in O(1) time, so outdegree is also O(1).</p>


<br/><br/>
<h3 id="outgoing">Outgoing</h3>

<p>Outgoing(v,c) returns the target node after traversing edge c from node v, which might not exist.
The hard part is finding the correct edge index to follow; after that we can conveniently use the forward() function we defined earlier.</p>

<p>To ease the explanation, consider a simple bit vector <code>00110100</code>. To count how many 1s there are up to and including the 7th position, we would use rank.
The answer is 3, but at this stage we still don&#8217;t know the position of the 3rd 1 (we can&#8217;t see the bitvector). In general, it may or may not be in the 7th position.
We could scan backwards, or we could just use select(3) to find the position (since this returns the first position i that has rank(i) = 3).</p>

<p>So essentially we can count how many of those edges there are before the node we are interested in, then use select to find the position. If the position
is inside this nodes range (of contiguous edges), then we follow it.</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/outgoing.png />
</figure>
</center>

<p>We complicated things a bit by separating our edges into flagged and non-flagged, so we may have to issue two of these queries (for the minus flags). The flagging
is useful later in <a href="#indegree">Indegree</a>.</p>

<p>In the next example, our nonflagged edge doesn&#8217;t fall in our nodes range, so we make a second query. It is possible that this one will return a positive result, but in the example
it doesn&#8217;t. By that stage though, we can respond with by returning &#8211;1 to signal that the edge doesn&#8217;t exist.</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/outgoing-neg.png />
</figure>
</center>

<p>forward() is defined to move us to the last edge of the resulting node, so the value in last will be 1. Hence, we can use rank to convert our edge index into a node
index before returning it.</p>

<p>This is a constant number of calls to O(1) functions, so outgoing is also O(1).</p>


<br/><br/>
<h3 id="label">Label</h3>

<p>At some point (e.g. during traversal) we are probably going to want to print the node labels out. Let&#8217;s work out how to do that (:</p>

<p>Remember, we aren&#8217;t storing the node labels explicitly. The F array will come in handy:
We can use the position of our node (found using select) as a reverse lookup into F. This can be done in constant time with a sparse bit vector,
or in logarithmic time using binary search, or we can use a linear scan if our alphabet is small.
In any case, lets assume it is O(1) time, although a linear scan might be faster in practice (fewer function calls may yield a lower constant coefficient).</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/last-symbol.png />
</figure>
</center>

<p>In the above example, we select to node 6 (the 7th 1 in last), which gives us the last edge index (any edge will do, but the last one is the easiest to find).
This happens to be row 8, so from F we know that all the last symbols between on the open range [7,10) are G.</p>

<p>Then we just use bwd() on the current edge to find an edge that pointed to this node, then rinse and repeat k times.</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/label.png />
</figure>
</center>


<br/><br/>
<h3 id="indegree">Indegree</h3>

<p>In a similar manner to <a href="#outdegree">outdegree</a>, all we need to do is count the edges that point to the current node label.
Take for example the graph:</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/indegree-graph.png />
</figure>
</center>

<p>We can easily find the first incoming edge by using backward(). To count the remaining edges our minus flags come in handy;
In the W array, the next G (non-flagged) belongs to a different node,
because we defined W to have minus flags if the source node has the same $k&#8211;1$ suffix (or, if same-labeled edges also share the same target node).</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/indegree1.png />
</figure>
</center>

<p>From here it is simple enough to do a linear scan (or even use select) until the next non-flagged edge; the maximum distance it could be is $\sigma^2$,
For larger alphabets, a more efficient method is to use rank instead:</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/indegree2.png />
</figure>
</center>

<p>First we find the position of the next non-flagged $G$, which gives us the end point of our range (we already have the start point from the first rank).
Then we use rank to calculate how many $G-$ there are to the end position, and subtract the initial rank value from this, giving us how many $G-$ occur within the range.</p>

<p>This is once again a constant number of $O(1)$ function calls, which means $indegree()$ is also $O(1)$.</p>

<br/><br/>
<h3 id="incoming">Incoming</h3>

<p>Incoming, which returns the predecessor node that begins with the provided symbol, is probably the most difficult operation to implement. However, it
does use approaches similar to the previous functions.</p>

<p>Consider this: from indegree(), we already know how to count each of the predecessor nodes. We can access these nodes if we instead use select to
iterate over the predecessor nodes, rather than using rank to simply count. Then, to disambiguate them by first character, we can use label().</p>

<p>A linear scan over the predecessors in this fashion would work, but for large alphabets we can use binary search (with a select call before each array access)
to support this in $O(k \log \sigma)$ time; $\log \sigma$ for
the binary search, where each access is a $O(1)$ select, followed by $O(k)$ to compute the label.</p>

<p>This is demonstrated in the example below:</p>

<center>
<figure>
<img src=https://alexbowe.s3.amazonaws.com/blog/debruijn/incoming.png />
</figure>
</center>

<br/><br/>
<h2 id="conclusion">Conclusion</h2>

<p>In conclusion, by using memory more efficiently, hopefully the cost of genome seqeuencing can be reduced, both proliferating the technology, but also giving way to
more advanced population analysis. To that end, we have described a novel approach to representing de Bruijn graphs efficiently, while supporting a full suite of
navigation operations quickly. Much of the (BWT-inspired) construction can be done efficiently on disk, but we intend to improve this soon to compete with Minia.</p>

<p>The total space is a theoretical $m(2 + \log{\sigma} + o(1))$ bits in general, or $4m + o(m)$ bits for DNA, given $m$ edges.
Using specially modified indexes we can lower this to  around 3 bits per edge.</p>

<p>I apologize that an efficient implementation isn&#8217;t available, nor have I provided experimental results. But if you found this post interesting
you can get your hands dirty with the <a href="https://github.com/alexbowe/debby/blob/0.1.1/debby.py">Python implementation I have provided</a>. The results and efficient implementation are on their way (:</p>

<p>And as always, follow me on <a href="http://www.twitter.com/alexbowe">Twitter</a>, and feel free to send me questions, criticisms, and especially pull requests!</p>

<div class="footnotes">
<hr />
<ol>

<li id="fn:1">
<p>Pevzner, P. A., Tang, H., and Waterman, M. S. (2001). An Eulerian path approach to DNA fragment assembly. Proceedings of the National Academy of Sciences, 98(17):9748–9753. <a href="#fnref:1" title="return to article" class="reversefootnote">&#160;&#8617;</a></p>
</li>

<li id="fn:2">
<p>Simpson, J. T., Wong, K., Jackman, S. D., Schein, J. E., Jones, S. J. M., and Birol, I. (2009). ABySS: A parallel assembler for short read sequence data. Genome Research, 19(6):1117–1123. <a href="#fnref:2" title="return to article" class="reversefootnote">&#160;&#8617;</a></p>
</li>

<li id="fn:3">
<p>Conway, T. C. and Bromage, A. J. (2011). Succinct data structures for assembling large genomes. Bioinformatics, 27(4):479–486. <a href="#fnref:3" title="return to article" class="reversefootnote">&#160;&#8617;</a></p>
</li>

<li id="fn:4">
<p>Okanohara, D. and Sadakane, K. (2006). Practical Entropy-Compressed Rank/Select Dictionary. <a href="#fnref:4" title="return to article" class="reversefootnote">&#160;&#8617;</a></p>
</li>

<li id="fn:5">
<p>R. Chikhi, G. Rizk. (2012) Space-efficient and exact de Bruijn graph representation based on a Bloom filter. WABI. <a href="#fnref:5" title="return to article" class="reversefootnote">&#160;&#8617;</a></p>
</li>

<li id="fn:6">
<p>M. Burrows and D. J. Wheeler. A Block-sorting Lossless Data Compression Algorithms. Technical Report 124, Digital SRC Research Report, 1994. <a href="#fnref:6" title="return to article" class="reversefootnote">&#160;&#8617;</a></p>
</li>

<li id="fn:7">
<p>P. Ferragina, F. Luccio, G. Manzini, and S. Muthukrishnan. Compressing and indexing labeled trees, with applications. Journal of the ACM, 57(1):4:1–4:33, 2009. <a href="#fnref:7" title="return to article" class="reversefootnote">&#160;&#8617;</a></p>
</li>

<li id="fn:8">
<p>The paper also describes an online construction method (where we can update by appending), and an iterative method that builds the k-dimensional de Bruijn
graph from an existing (k&#8211;1)-dimensional de Bruijn graph. <a href="#fnref:8" title="return to article" class="reversefootnote">&#160;&#8617;</a></p>
</li>

<li id="fn:9">
<p>We currently use an external merge sort, but intend to optimise as this is where Minia beats us time-wise. <a href="#fnref:9" title="return to article" class="reversefootnote">&#160;&#8617;</a></p>
</li>

<li id="fn:10">
<p>This is &#8220;little o&#8221; notation, which may be unfamiliar to some people. Intuitively it means &#8220;grows much slower than&#8221;, and is stricter than big O.
A formal definition can be found on <a href="https://en.wikipedia.org/wiki/Big_O_notation#Little-o_notation">Wikipedia</a>. <a href="#fnref:10" title="return to article" class="reversefootnote">&#160;&#8617;</a></p>
</li>

<li id="fn:11">
<p>http://stackoverflow.com/questions/1801135/what-is-the-meaning-of-o-polylogn-in-particular-how-is-polylogn-defined <a href="#fnref:11" title="return to article" class="reversefootnote">&#160;&#8617;</a></p>
</li>

<li id="fn:12">
<p>R. Raman, V. Raman, and S. R. Satti. Succinct indexable dictionaries with applications to encoding k-ary trees, prefix sums and multisets. ACM Trans. Algorithms, 3(4), November 2007. <a href="#fnref:12" title="return to article" class="reversefootnote">&#160;&#8617;</a></p>
</li>

<li id="fn:13">
<p>P. Ferragina, G. Manzini, V. M ̈akinen, and G. Navarro. Compressed Representations of Sequences and Full-Text Indexes. ACM Transactions on Algorithms, 3(2):No. 20, 2006. <a href="#fnref:13" title="return to article" class="reversefootnote">&#160;&#8617;</a></p>
</li>

</ol>
</div>