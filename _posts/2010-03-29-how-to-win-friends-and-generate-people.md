---
ID: 199
post_title: How to Win Friends and Generate People
author: Alex
post_excerpt: ""
layout: post
permalink: >
  https://alexbowe.com/how-to-win-friends-and-generate-people/
published: true
post_date: 2010-03-29 05:47:36
---
<img src="https://alexbowe.com/wp-content/uploads/2013/01/2859_largeview.jpg" alt="Dice" width="800" height="590" class="alignnone size-full wp-image-200" />
<p>I'm doing a project for a subject at RMIT which needs to manage thousands of patient records for a hospital. We haven't been given any sample data though, so I wanted to write a generator (so we can test it with small or large data sets whenever needed).</p>
<p>I started with the name generator (in Python), which selected a random male/female/last name from a file [1]. I then realised an address generator would behave similarly (street, city, country lists), so I decided to make a Generator base class. You can get the source code for these <a href="http://github.com/alexbowe/generators">here</a>.</p>
<p>I wanted the Generator base class to create methods dynamically; It would be instantiated with a bunch of methodname-filename pairs and have the methods for randomly selecting an entry from each file made dynamically. Exactly how to do this wasn't easy to find, as it wasn't well documented... the solution ended up being:</p>

https://gist.github.com/346711

<p><a href="http://niki.code-karma.com/">Niki</a> (my brother) helped me get 70% of the way there... (after lots of scope and decorator issues) thanks man :)</p>
<p>[1] The name lists were taken from <a href="http://www.census.gov/genealogy/www/data/1990surnames/names_files.html">this census data</a>, which actually provides percentages for each name too, if you wanted to make the name distribution more realistic...</p>