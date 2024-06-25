---
layout: post
title:  "My first ever Google Interview"
date:   2018-12-19
description: "I got a chance to interview for an SWE intern role at Google, India!"
tag:
- Google 
- technology
- interview
- jekyll
thumbnail: http://content.timesjobs.com/img/61544867/Master.jpg
giscus_comments: true
---


Hey! So I'm currently in my 3rd year right now. In our college, we bag summer internships in different companies in the starting of our 5th semester. I bagged a software engineering intern at an Indonesian startup called Go-Jek. I do know that its a good place to spend my summer there because:

1. Since its a startup, the culture is going to be work-intensive (which I think I'll love).
2. Go-Jek  has a special bootcamp desgined for all of its new developers, to make them learn coding and testing in an organised and correct way.
3. If I get a PPO (Pre Placement Offer), its highly likely I get to visit Singapore, Indonesia, and surrounding countries on a daily basis :3 .
4. Well, also, they pay well :P .

But still, I had been applying to a *lot* of other places because, you know, I'm allowed to :P . And I wasnt applying to just internship roles. I was filling out forms for various scholarships too! So now you might think that here I am, so actively engaging myself in different opprotunities, something good must have happened to me for sure. Well, yeah!

1. I got rejection letters from all the scholarship programmes.
2. I never heard back from 99% of the internship roles I had applied to.
3. The ones that did respond back told me I wasn't competent enough. :D

Obviously, this was a very effective way to break my spirit (not completely though, but certainly made a crack). Then one day unexpectedly I receive this mail:

![Whoa](../assets/img/google.png)

Ummm, ALRIGHT?!

The first thing I did was call my mom and she said its probably a hoax/prank email(Thanks for the motivation, mom). I had never heard of people bagging interviews this way, so obviously all of this was a bit suspicious. Well, I ended up stalking the recruiter on LinkedIn. She seemed pretty legit. So I sent her a message on LinkedIn and got no response there. Waited patiently for a few days, and got a mail about confirming my interview dates. So all this was **really** happening! :O And all of this wasn't even just any company. IT WAS GOOGLE OMG. 

I thought to myself:

> This was it. This is my big break.

The recruiter sent me study resources. I had my end semester exams going on so I only got to practice for two days. I covered all the basic data structure topics.

_And the calls came._ Dot on time too. So I had 2 telephonic interviews, both of 45 min with a gap of 15 min.

The 1st call was from Google, Singapore. I could tell he was a South Indian. He asked me about myself and asked the following 2 questions:

> 1. **Given an integer array, count the number of ways you can divide it into 3 contiguous arrays of equal sums.**

I solved this in O(n^2) time complexity and O(1) space complexity.

> 2. **Given a complete binary tree and an element value, return the node containing that element.**

I solved this is O(log(n)) time complexity and O(log(n)) space complexity.

He was patient and gave me 1 hint each for both the questions. This interview got extended upto 61 minutes :3.

The 2nd call was from Google,Hyderabad. He too was a South Indian. He jumped straight to the questions:

> 1. **Given 2 keypresses like [A-Za-z0-9] and backticks(\`) in an array, tell if the resultant strings would be equal**

I solved this in O(n) time complexity and O(1) space complexity.

> 2. **Given an array of integers, find all equivalence points where sum of array before the equivalence point = sum of array after the equivalence point.**

I solved this is O(n) time complexity and O(n) space complexity.

I solved both the questions with no hints. This interview got extended upto 57 minutes :3.


3 days later I got a call from my recruiter telling me I have another interview lined up for next week (Yay?). So, this time, I studied *hard*. Solved all kinds of logical questions, graph traversals, greedy solutions, etc. I was prepared to go for it with all my effort.

_And the call came again._ Five minutes late this time. There was just 1 telephonic call of 45 min (actually 39 min). He called from Google, Hyderabad.

> 1. **Given a very large file of entries like: 
        USD, EUR, 0.69
        EUR, YEN, 1.6
        YEN, RUB, 2.4
        find answer of queries like: USD,YEN,_?**

It was a pretty vague question. He didn't tell me much about the format of the file or anything related. I told him I'd solve this using DFS. I explained the logic. He seemed satisfied. He asked me to code it. I asked him to give me an idea of how its stored so that I could manipulate it into data structures. He gave me this:

`vector<vector<string>>conversion_rates`

So the only efficient way to manipulate this according to my logic was:

`map<string,map<string,float>>rate_list`

*sigh*

I got heavily confused with the iterators of this STL. Got nervous, my mind stopped working and basically couldn't code it up much. He told me I needed practice and cut the phone. I felt super dejected :P 
Well, its been 2 days to that now, and though I do feel as if I lost a great opportunity, I'm thoroughly greatful for this experience. I also hope (desperately) that something great happens soon, cuz I could do with some morale boosting amidst all these collective rejections xD

Anyway, this was my Google story. What's yours? :)  
