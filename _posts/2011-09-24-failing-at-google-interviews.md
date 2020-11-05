---
ID: 57
post_title: Failing at Google Interviews
author: Alex
post_excerpt: ""
layout: post
permalink: >
  https://alexbowe.com/failing-at-google-interviews/
published: true
post_date: 2011-09-24 07:18:41
---
I've participated in about four <em>sets</em> of Google interviews (of about 3 interviews each) for various positions. I'm still not a Googler though, which I guess indicates that I'm not the best person to give this advice. However, I think it's about time I put in my 0.002372757892 bitcoins. I recently did exactly this to help my brother prepare for his interviews and the guy kicked ass. If he gets the job I'm going to take as much credit for it as I can ;)

During my interviews I didn't sign a NDA, but I do respect <a href="http://igor.moomers.org/interview-questions/">the effort that interviewers put into preparing their questions</a> so I'm not going to discuss them. That doesn't matter though, because you probably won't get the same questions anyway, and the algorithm stuff is far from the whole story.

This post is mainly about the rituals I perform during preparation for the interviews, and the lessons I have learned from them. I am of the strong opinion that everyone should apply for a job at Google.

<h2>Why Should I?!</h2>

Not everyone wants to work for Google, but there are valuable side effects to a Google interview. Even if you don't think you want a job there, or think that you are under-qualified, it is a great idea to just try for one. The absolute worst thing that could happen is that you have fun and learn something.

A couple of the things I learned are algorithms for (weighted) random sampling, queueing, vector calculus, and some cool applications of bloom filters.

The people you will talk to are smart, and it's a fun experience to be able to solve problems with smart and passionate people. One of my interviews was just a discussion about the good and bad parts (in our opinions) of a bunch of programming languages (Scheme, Python, C, C++, Java, Erlang). We discussed <a href="http://mitpress.mit.edu/sicp/">SICP</a> and the current state of education, and he recommended some research papers for me to read. All the intriguing questions and back-and-forth made me feel like I was being taught by a modern <a href="http://en.wikipedia.org/wiki/Socratic_method">Socrates</a> (perhaps Google should consider offering a Computer Science degree taught entirely with interviews :P).

Sadly, a subsequent interview stumped me because I didn't understand the requirements. Even the stumping interviews have given me a great chance to realise some gaps in my knowledge and refine my approach. I knew that it was important to get the requirements right, but this really drove it home.

I hope I've got you curious about what you could learn from a Google interview. If you are worried about the possible rejection, treat it as a win in a game of <a href="http://rejectiontherapy.com/rules/">Rejection Therapy</a>. You can re-apply as many times as you like, so you could also think of it as <a href="http://en.wikipedia.org/wiki/Test-driven_development">TDD</a> for your skills, and you like TDD, right?

<h2>How To Prepare for the Interview : Technical</h2>

When you are accepted for a phone interview, Google sends you an email giving you tips on how to prepare. Interestingly, this has been a different list each time. I'll discuss the one I liked the most. They only give advice on the technical side. I will also discuss what I think are some other important aspects to be mindful of.

First of all, you are going to want to practice. Even if you have been coding every day for years, you might not be used to the short question style. <a href="http://projecteuler.net/">Project Euler</a> is <em>the bomb</em> for this. You will learn some maths too, which will come in handy, and it builds confidence. Do at least one of these every day until your interview.

You will also want some reading material. Google recommended <a href="http://steve-yegge.blogspot.com/2008/03/get-that-job-at-google.html">this post by Steve Yegge</a>, which does a good job of calming you. They also recommended <a href="http://sites.google.com/site/steveyegge2/five-essential-phone-screen-questions">another post by Steve Yegge</a> where he covers some styles of questions that are likely to be asked. Yegge recommends a particular book very highly - <a href="http://www.algorist.com/">The Algorithm Design Manual</a>:

<img style="float: right; margin: 10px 50px 10px 50px;" src="http://ecx.images-amazon.com/images/I/71LzXKygXpL.jpg" alt="" width="180px" />

<blockquote>
  More than any other book it helped me understand just how astonishingly commonplace (and important) <strong>graph problems</strong> are – they should be part of every working programmer's toolkit. The book also covers basic data structures and sorting algorithms, which is a nice bonus. But the gold mine is the second half of the book, which is a sort of encyclopedia of <strong>1-pagers</strong> on zillions of useful problems and various ways to solve them, without too much detail. Almost every 1-pager has a simple picture, making it easy to remember. This is a great way to learn how to <strong>identify hundreds of problem types</strong>.
</blockquote>

I haven't read the whole thing, but what I have read of it is eye and mind opening. This wasn't recommended to me directly by Google recruiting staff, but one of my interviewers emailed me a bunch of links after, including a link to the page for this book. There was a recent review of this book featured on <a href="http://eriwen.com/books/best-algorithms-book/">Hacker News</a>. It is very good. The author, Steve Skiena, also offers his <a href="http://www.cs.sunysb.edu/~algorith/video-lectures/">lecture videos and slides</a> - kick back and watch them with a beer after work/uni.

<img style="float: left; margin: 10px 50px 10px 50px;" src="http://ecx.images-amazon.com/images/I/41WonSY9PbL._SX258_BO1,204,203,200_.jpg" alt="" width="180px" />

If the size of The Algorithm Design Manual is daunting and you want a short book to conquer quickly (for morale reasons), give <a href="http://cm.bell-labs.com/cm/cs/pearls/">Programming Pearls</a> a read. Answer as many questions in it as you can.

Additionally, <a href="https://www.interviewcake.com/">Interview Cake</a> offers a new approach, which systematises your technical preparation so you can know exactly what to focus on while avoiding becoming overwhelmed. It is a paid service, but they also have a free mailing list with weekly questions to keep you sharp (great for your long-game).

The phone interviews usually are accompanied by a Google doc for you to program into. I usually nominate Python as my preferred language, but usually they make me use C or C++ (they often say I can use Java too). I was rusty with my C++ syntax at the time, but they didn't seem to mind. I just explained things like using templates, even though I can never remember the syntax for the cool <a href="http://www.amazon.com/Modern-Design-Generic-Programming-Patterns/dp/0201704315">metaprogramming tricks</a>.

Speaking of tricks, you get style points for using features of the language that are less well known. I had an interviewer say he was impressed because I used Pythons pattern matching (simple example: <code>(a, b) = (b, a)</code>). List comprehensions, map/reduce, generators, lambdas, and decorators could all help make you look cool, too. Only use them if they are <em>useful</em> though!

<h2>How To Prepare for the Interview : Non-Technical</h2>

There will also be a few non-technical questions. When I did my first one, a friend recommended that I have answers ready for cookie-cutter questions like <em>"Where do you see yourself in ten years?"</em> and <em>"Why do you want to work for Google?"</em>. Don't bother with that! Do you really think one of the biggest companies in the world will waste their time asking questions like that? Every candidate would say the same answer, something about leading a team and how Google would let you contribute to society, or whatever (great, but <em>everyone</em> wants that).

They <em>will</em> ask you about your previous work and education, though, and pretty much always ask about a technical challenge you overcame. I like to talk about a fun incremental A* search I did at my first job (and why we needed it to be iterative). You can probably think of something, don't stress, but better to think of it before the interview.

And have a question ready for when they let you have your turn. Don't search for <em>"good questions to ask in technical interviews"</em>, because if it isn't <em>your</em> question, you might be uninterested if the interviewer talks about it for a long time. Think of something that you could have a discussion about, something you are opinionated about. Think of something you hated at a previous job (but don't come across as bitter), how you would improve that, and then ask them if they do that. For me, I was interested in the code review process at Google, and what sort of project they would assign to a beginner.

I know someone who asked questions from <a href="http://www.joelonsoftware.com/articles/fog0000000043.html">The Joel Test</a>. The interviewer might recognise these questions and either congratulate you on reading blogs about your field, or quietly yawn to themselves. It's up if you want to take that risk (well, it's not a <em>big</em> risk). I definitely think it's better to ask about something that has the potential to annoy you on a personal level if they don't give you the answer you want ;) it's subtle, but people can detect your healthy arrogance and passion.

If you have a tech blog, refer to it. I've had interviewers discuss my posts with me (which they found from my resume). Blogs aren't hard to write, and even a few posts on an otherwise barren blog will make you look more thoughtful.

Finally, the absolute best way to prepare for a Google interview is to do more Google interviews, so if you fail, good for you! ;)

<h2>Just Before the Interview</h2>

Here are a few things that help me handle the pressure before an interview.

One time I was walking to an interview in the city (not a Google interview) and I was really nervous, even though I didn't care either way if I got the job. I thought about how the nerves wouldn't be an issue after the interview, because I'd have already done the scary thing by then. I couldn't time travel, but I instead wondered if there is a way to use up the nerves on something else.

There was a girl walking next to me, so I turned to her and said she was dressed nicely. She said a timid "thank you" and picked up pace to get away from me. I laughed at my failure, but suddenly I didn't feel so scared about the interview. I think this is a great example of why <a href="http://rejectiontherapy.com/rules/">Rejection Therapy</a> is worth experimenting with.

So yeah, talk to a stranger. If you are waiting at home for a phone call though, another thing I do is jack jumps, dancing, or jogging on the spot just to make myself forget the other reason my heart is pounding so fast.

<h2>During the Interview</h2>

If you are doing a phone interview, answer it standing up (you can sit down after) and pace around a little bit. Smile as you talk, as well. You should also take down their name on paper ready to use a few times casually. These are tricks from the infamous <a href="http://en.wikipedia.org/wiki/How_to_Win_Friends_and_Influence_People">How to Win Friends and Influence People</a>. Maybe <em>these alone</em> won't make you likeable, but I think it causes you to think about the other person and stop being so self conscious, which helps you to relax. You'll be <a href="http://www.youtube.com/watch?v=u8UE4P8kB-c">one charming motherfucking pig</a>.

Take some time to think before answering, and especially to seek clarification on the questions. Ask what the data representation is. I've found that they tend to say "whatever you want". In a graph question, I said "Okay, then it's an adjacency matrix", which made the question over and done with in ten seconds. The interviewer seemed to like that, so don't be afraid to be a (humble) smart ass.

You might recognise the adjacency matrix as potentially being a very poor choice, depending on the nature of the graph. I did discuss when this might not be a good option. In fact, for every question, I <strong>start off by describing a naive approach</strong>, and then refine it. This helps to verify the question requirements, and gives you an easy starting point. Maybe you could introspectively comment on agile methodology (Google practises Scrum).

One last thing! Google schedules the interview to be from 45 minutes to an hour. I have had awkward moments at the end of interviews where the interviewer mentions that our time is nearly up, and <em>then</em> asks another question, or asks if I have any questions. It made me feel like he was in a rush, so I didn't feel like expanding on things much. Now, I recommend taking as much time as they will give you. Keep talking until they hang up on you if you have to :) although it might help to say "I don't mind if we go over, as long as I'm not keeping you from something" when the interviewer mentions the time.

<h2>Reflect</h2>

<a href="http://steve-yegge.blogspot.com/2008/03/get-that-job-at-google.html">Steve Yegge</a> says there are lots of smart Googlers who didn't get in until their third attempt (I still haven't gotten in after my fourth, and I don't think I'm stupid). As I mentioned, I'm writing this post because I found the process of doing a Google interview at all to be very rewarding.

It is important to reflect afterwards in order to reap the full benefits of interviewing at Google. If you did well, why? But more importantly, if you feel you did poorly, why? Google won't give feedback, which can be a bit depressing at times. After each interview write notes about what you felt went well and what didn't - this way you can look back if you don't get the job, and decide what you need to work on. This post is the culmination of my reflections and the notes - if you decide to write a blog post, I'd enjoy reading it and will link it here.

If you want more blog posts to read about how to get better at Computer Science, I recently found <a href="http://matt.might.net/articles/what-cs-majors-should-know/">this post by Matt Might</a> to be a good target to aim for. Check out <a href="http://profshonle.blogspot.com/2010/08/ten-things-every-computer-science-major.html">Ten Things Every Computer Science Major Should Learn by Macneil Shonle</a> as well, and my previous post <a href="https://alexbowe.com/advice-to-cs-undergrads">Advice to CS Undergrads</a> (the links at the end in particular).

And as always, please read the comments below and add your own thoughts to the discussion. In particular, <a href="https://alexbowe.com/failing-at-google-interviews/#comment-917263867">Sumit Arora</a> gave some important advice that I didn't cover.

Have you really read this far? Consider adding me to <a href="http://www.twitter.com/alexbowe">Twitter</a> and telling me what you thought :)