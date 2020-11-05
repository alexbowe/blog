---
ID: 409
post_title: DRY Function Pointers in C
author: Alex
post_excerpt: ""
layout: post
permalink: >
  https://alexbowe.com/dry-function-pointers-in-c/
published: true
post_date: 2013-09-09 14:40:28
---
Just a quick post today about C function pointers. Over the past two years I have seen the occasional function pointer introduction post on <a href="http://news.ycombinator.com">Hacker News</a>, but I rarely see <a href="https://twitter.com/MomAdviceBot">this one weird trick</a>.

The most recent I have read was <a href="http://denniskubes.com/2013/03/22/basics-of-function-pointers-in-c/">this one by Dennis Kubes</a>. I haven't hung out with C for a while (I more frequently visit its <a href="http://harmful.cat-v.org/software/c++/">octopussier cousin</a>, onto which a lambda-shaped leg has been nailed), so the post triggered some fond memories.

This post is just a leg nailed to his post, so go read it now if you would like to know about function pointers in C.

Kubes presents an example where he defines a <code>domath</code> function, that accepts a pointer to a function that performs an operation on two integers:

<pre><code>int domath(int (*mathop)(int, int), int x, int y) {
  return (*mathop)(x, y);
}
</code></pre>

For a more realistic example, let's say you implemented <a href="http://en.wikipedia.org/wiki/Fold_(higher-order_function)">fold</a> instead, and hey, why not left-fold also?

<pre><code>int fold_right(int (*f)(int, int), int init, const int * first, const int * last) {
  int result = init;
  while(first != last) {
    result = f(result, *first++);
  }
  return result;
}

int fold_left(int (*f)(int, int), int init, const int * first, const int * last) {
  int result = init;
  while(first != last) {
    result = f(*last--, result);
  }
  return result;
}
</code></pre>

Now imagine a whole family of higher order functions like these... the <em>two</em> function signatures here are bad enough already. The syntax doesn't communicate well - you actually have to work out wtf <code>int (*f)(int, int)</code> means, unless you write C in your sleep. Even then, there is a chance the next person to maintain your code will cry.

Let's look out for our fellow coder and <a href="http://en.wikipedia.org/wiki/Dont_repeat_yourself"><strong>DRY</strong></a> those tears. C has syntax for <code>typedef</code>ing function pointers. The result is much <a href="http://www.youtube.com/watch?v=D28IN9ogoNo">friendlier</a>:

<pre><code>typedef int (*binary_int_op) (int, int);

int fold_right(binary_int_op f, int init, const int * first, const int * last) {
  int result = init;
  while(first != last) {
    result = f(result, *first++);
  }
  return result;
}

int fold_left(binary_int_op f, int init, const int * first, const int * last) {
  int result = init;
  while(first != last) {
    result = f(*last--, result);
  }
  return result;
}
</code></pre>

That is still some weird ass syntax at first, but you write that once (per type/intent of function pointer) and then live a happier life. And if you choose the name intelligently, you can <strong>communicate the intent</strong> as fast as the reader can read English.

Or you could ditch C and use <a href="http://www.dreamincode.net/forums/topic/264061-c11-fun-with-functions/">C++'s functional programming tools</a>.