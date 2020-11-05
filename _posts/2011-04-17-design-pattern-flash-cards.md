---
ID: 136
post_title: Design Pattern Flash Cards
author: Alex
post_excerpt: ""
layout: post
permalink: >
  https://alexbowe.com/design-pattern-flash-cards/
published: true
post_date: 2011-04-17 22:03:04
---
<img src="https://alexbowe.com/wp-content/uploads/2011/04/flashcards.png" alt="flashcards" width="675" height="388" class="aligncenter size-full wp-image-286" />
<p>Last year I studied a subject which required me to memorise design patterns. I tried online flash card web sites, but I was irritated that I didn't own the data I put up (they had no export option). So I wrote a something in Python to generate flash cards for me using LaTeX and the Cheetah templating library. The repository is hosted <a href="https://github.com/alexbowe/cardgen">here</a>, although it could do with a refactor.</p>
<p>If you don't want to generate your own, you can download the pre-generated design pattern intent flash cards <a href="https://github.com/alexbowe/cardgen/raw/master/intents.pdf">here</a> which contains the 23 original design patterns from the Gang Of Four.</p>
<p>To generate your own flash cards, create an input text file with this structure:</p>
<pre><code>Front text (such as pattern name):
Definition line 1.
Definition line 2.
</code></pre>
<p>For example:</p>
<pre><code>Abstract Factory:
Provides an interface for creating families of related or
dependent objects without specifying their concrete classes.
</code></pre>
<p>Currently the front text is single-line only. The regex could be updated of course (if you do, feel free to send a pull request!).</p>
<p>To compile this:</p>
<pre><code>./cardgen.py -i inputfile -o outputfile
pdflatex outputfile
</code></pre>
<p>Then just print it out on a double side printer (or glue the two sheets together). I carried these around with me all the time during the lead-up to the exam, and I was scay-fast when it came to recalling which design pattern did what. Just flick through them (shuffle first) in forward or reverse order when you are on the train next :)</p>