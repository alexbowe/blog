---
ID: 142
post_title: 'Au Naturale &#8211; an Introduction to NLTK'
author: Alex
post_excerpt: ""
layout: post
permalink: https://alexbowe.com/au-naturale/
published: true
post_date: 2011-03-20 15:14:31
---
<p><img src="http://data.whicdn.com/images/1396251/tumblr_kwukdtbqLx1qargqko1_400_large.jpg " width="400" height="400" class="aligncenter" />
This blog post is an introduction on how to make a key phrase extractor in Python, using the <a href="http://www.nltk.org/">Natural Language Toolkit (NLTK)</a>.</p>

But how will a search engine know what it is about? How will this document be indexed correctly? A human can read it and tell that it is about programming, but no search engine company has the money to pay thousands of people to classify the entire Internet for them. Instead they must reasonably predict what a human may decide to be the key points of a document. And they must automate this.

Remember how proper sentences need to be structured with a <a href="http://hubpages.com/hub/Grammar_Mishaps__Building_a_Sentence">subject and a predicate</a>? A subject could be a noun, or a adjective followed by a noun, or a pronoun... A predicate may be or include a verb... We can take a similar approach by defining our key phrases in terms of what types of words (or parts-of-speech) they are, and the pattern in which they occur.

But how do we know what words are nouns or verbs in an automated fashion?

Throughout this post I will use an excerpt from <a href="http://www.amazon.com/gp/product/0061673730/alexbowecom-20">Zen and the Art of Motorcycle Maintenance</a> as an example:

<blockquote>
  The Buddha, the Godhead, resides quite as comfortably in the circuits of a digital computer or the gears of a cycle transmission as he does at the top of a mountain or in the petals of a flower. To think otherwise is to demean the Buddha...which is to demean oneself.
</blockquote>

Before proceeding, make a (mental) note of the key phrases here. What is the document about?

<h2>Tokenizing</h2>

In a program, text is represented as a string of characters. How can we go about moving <em>one</em> level of abstraction up, to the level of words, or <em>tokens</em>? To tokenize a sentence you may be tempted to use Python's <code>.split()</code> method, but this means you will need to code additional rules to remove hyphens, newlines and punctuation when appropriate.

Thankfully the <a href="http://www.nltk.org/">Natural Language Toolkit (NLTK)</a> for Python provides a regular expression tokenizer. There is an example of it (including how it fares against Pythons regular expression tokenization method) in <a href="http://nltk.googlecode.com/svn/trunk/doc/book/ch03.html">Chapter 3</a> of the <a href="http://www.nltk.org/book">NLTK book</a>. It also allows you to have comments:

<pre><code># Word Tokenization Regex adapted from NLTK book
# (?x) sets flag to allow comments in regexps
sentence_re = r'''(?x)
  # abbreviations, e.g. U.S.A. (with optional last period)
  ([A-Z])(\.[A-Z])+\.?
  # words with optional internal hyphens
  | \w+(-\w+)*
  # currency and percentages, e.g. $12.40, 82%
  | \$?\d+(\.\d+)?%?
  # ellipsis
  | \.\.\.
  # these are separate tokens
  | [][.,;"'?():-_`]
'''
</code></pre>

Once we have constructed our regex for defining what sort of format our words should be in, we call it like so:

<pre><code>import nltk
#doc is a string containing our document
toks = nltk.regexp_tokenize(doc, sentence_re)

&gt;&gt;&gt; toks
['The', 'Buddha', ',', 'the', 'Godhead', ',', 'resides', ...
</code></pre>

<h2>Tagging</h2>

The next step is tagging. This uses statistical data to apply a Part-of-speech tag to each token, e.g. ADJ, NN (Noun), and so on. Since it is statistical, we need to either train our model or use a pre-trained model. NLTK comes with a pretty good one for general use, but if you are looking at a certain kind of document you may want to train your own tagger, since it may greatly affect the accuracy (think about very vocabulary-dense fields such as biology).

Note that to train your own tagger you will need a pre-tagged corpus (NLTK comes with some) or use a <em>bootstrapped</em> method (which can take a long time). Check out  <a href="http://streamhacker.com/2010/04/12/pos-tag-nltk-brill-classifier/">Streamhacker</a> and <a href="http://nltk.googlecode.com/svn/trunk/doc/book/ch05.html">Chapter 5 of the NLTK book</a> for a good discussion on training your own (and how to test it empirically).

For the sake of this introduction, we will use the default one. The result is a list of token-tag pairs:

<pre><code>&gt;&gt;&gt; postoks = nltk.tag.pos_tag(toks)
&gt;&gt;&gt; postoks
[('The', 'DT'), ('Buddha', 'NNP'), (',', ','), ('the', 'DT'), ...
</code></pre>

<h2>Chunking</h2>

Now we can use the part-of-speech tags to lift out noun phrases (NP) based on patterns of tags.

<img src="http://nltk.googlecode.com/svn/trunk/doc/images/chunk-segmentation.png" width="600" height="405" class="aligncenter" />

<em>Note: All diagrams have been stolen from the NLTK book (which is available under the Creative Commons Attribution Noncommercial No Derivative Works 3.0 US License).</em>

This is called chunking. We can define the form of our chunks using a regular expression, and build a chunker from that:

<pre><code># This grammar is described in the paper by S. N. Kim,
# T. Baldwin, and M.-Y. Kan.
# Evaluating n-gram based evaluation metrics for automatic
# keyphrase extraction.
# Technical report, University of Melbourne, Melbourne 2010.
grammar = r"""
    NBAR:
        # Nouns and Adjectives, terminated with Nouns
        {&lt;NN.*|JJ&gt;*&lt;NN.*&gt;}

    NP:
        {&lt;NBAR&gt;}
        # Above, connected with in/of/etc...
        {&lt;NBAR&gt;&lt;IN&gt;&lt;NBAR&gt;}
"""

chunker = nltk.RegexpParser(grammar)
tree = chunker.parse(postoks)
</code></pre>

It is also possible to describe a Context Free Grammar (CFG) to do this, and help deal with ambiguity - information can be found in <a href="http://nltk.googlecode.com/svn/trunk/doc/book/ch08.html">Chapter 8 of the NLTK book</a>. Chunk regexes can be much more complicated if needed, and support <em>chinking</em>, which allows you to specify patterns in terms <em>what you don't want</em> - see <a href="http://nltk.googlecode.com/svn/trunk/doc/book/ch07.html">Chapter 7 of the NLTK book</a>.

The output of chunking is a tree, where the noun phrase nodes are located just one level before the leaves, which are the words that constitute the noun phrase:

<img src="http://nltk.googlecode.com/svn/trunk/doc/images/chunk-treerep.png" width="600" height="696" class="aligncenter" />

To access the leaves, we can use this code:

<pre><code>def leaves(tree):
    """Finds NP (nounphrase) leaf nodes of a chunk tree."""
    for subtree in tree.subtrees(filter = lambda t: t.node=='NP'):
        yield subtree.leaves()
</code></pre>

<h2>Walking the tree and Normalisation</h2>

We can now walk the tree to get the terms, applying normalisation if we want to:

<pre><code>def get_terms(tree):
    for leaf in leaves(tree):
        term = [ normalise(word) for word, tag in leaf
            if acceptable_word(word) ]
        yield term
</code></pre>

Normalisation may consist of lower-casing words, removing stop-words which appear in many documents  (i.e. if, the, a...), stemming (i.e. cars $&#92;rightarrow$ car), and lemmatizing (i.e. drove, drives, rode $&#92;rightarrow$ drive). We normalise so that at later stages we can compare similar key phrases to be the same; <code>'the man drove the truck'</code> should be comparable to <code>'The man drives the truck'</code>. This will allow us to better rank our key phrases :)

Functions for normalising and checking for stop-words are described below:

<pre><code>lemmatizer = nltk.WordNetLemmatizer()
stemmer = nltk.stem.porter.PorterStemmer()

def normalise(word):
    """Normalises words to lowercase and stems and lemmatizes it."""
    word = word.lower()
    word = stemmer.stem_word(word)
    word = lemmatizer.lemmatize(word)
    return word

def acceptable_word(word):
    """Checks conditions for acceptable word: length, stopword."""
    from nltk.corpus import stopwords
    stopwords = stopwords.words('english')

    accepted = bool(2 &lt;= len(word) &lt;= 40
        and word.lower() not in stopwords)
    return accepted
</code></pre>

And the result is:

<pre><code>&gt;&gt;&gt; terms = get_terms(tree)
&gt;&gt;&gt; for term in terms:
...    for word in term:
...        print word,
...    print
buddha
godhead
circuit
digit comput
gear
cycl transmiss
mountain
petal
flower
buddha
demean oneself
</code></pre>

Are these similar to the key phrases you chose? There are lots of areas above that can be tweaked. Let me know what you come up with :) (the code can be found in <a href="https://gist.github.com/879414">this gist</a>).

In future posts I will talk about how to rank key phrases. I will also discuss how to scale this to process many documents at once using MapReduce.

In the mean time check out the <a href="http://text-processing.com/demo/">demos</a> on <a href="http://streamhacker.com/">Streamhacker</a>, solve the problems in the <a href="http://www.nltk.org/">NLTK book</a>, or read the <a href="http://www.amazon.com/gp/product/1849513600/alexbowecom-20">NLTK Cookbook</a> :)