---
layout: page
title: Readings
permalink: /readings/
---

This page will be used to periodically post papers on various topics that I'm reading each week or so.

Although I haven't really set any hard metrics for evaluation, I make an honest effort to learn at least a handful of new things each week.  So far I've found the totally manufactured pressure of trying to update my blog once a week does a pretty good job of motivating me to look for new data sets, new modeling techniques, or new code packages/languages/functionalities.  Basically, it helps me find new things to DO.

In addition to learning how to do new things, it's also (I think) pretty important to expose one's self to new ideas regularly.  In this regard I have set more concrete goals for myself.  In addition to any readings or resources I need to consult in the natural course of doing my job, I aim to: 

* read at least one recent peer reviewed paper from a top tier journal (American Economic Review, Journal of Economic Perspectives, Science, Applied Ecology, etc.) or working paper from a recent Ph.D (the new Ph.Ds are usually pretty plugged in to the 'hot' topics) 
* read at least one book chapter
* read/peruse one software development tutorial/technical manual/GitHub repo.

I'm not always great about meeting these goals....so I'm hoping a little more manufactured pressure in the form of of a periodic 'Readings' update will give me the same kick-in-the-ass I get from the blog.

Anyway, here's what I've been reading this week:

## Economics

Two papers from a collegaue at the [University of Dayton, Nancy Haskell](https://sites.google.com/site/nancylhaskell/home/research):

* The Pull of Popularity: Explaining Conformity in Student Behaviors
* Overcompensating for Better or for Worse: Effects of Being Racially, Ethnically, & Socioeconomically Different from Peers 

There is a really interesting literture in Economics (that I don't know nearly enough about) that seeks to better understand how we form relationships and the formation and maintainence of those relationships influences various behaviors and outcomes in our lives. 

## Modeling

[Discovering governing equations from data by sparse identification of nonlinear dynamical systems](http://www.pnas.org/content/113/15/3932.abstract)

If I understand this literature right I think it has some mind-blowing potential.  Economics, like most other sciences, has developed a set of theories for how people behave (firms maximize profits subject to budgetary and other constraints, individuals maximize satisfaction subject to budetary and other contraints).  When fitting models to data, these theories generally provide some guidance on what is and is not allowed with repect to model fitting.  The natural sciences, similarly, have rules governing how processes evolve:

* water flows according to certain physical rules
* fish grow according to certain systems of dynamic differential equations

Theories, by their nature, make predictions about outcomes one should expect given certain initial conditions.  In science, theories are often tested by matching observed data to the predictions made by the theory to see how 'well' a theory describes actual outcomes.  The literature addressed by the PNAS paper I linked to above takes a different tactic.  It starts with, for all intents and purposes, with a big bag of data and asks whether some coherant underlying data generating process can be recovered from the coupled dynamics present in the big bag of data. 


**Side Note**: Many people unfamiliar with how science combines theory and evidence mistakenly believe that these theories impose rigid constraints on the process of fitting models to data (in practice the mathematical concept of flexible functional forms is generally leveraged to theory to provide good first or second order approximations).  

Although (in my mind at least) having a firm theoretical foundation to one's empirical work is a good thing, I can see a lot of value in the idea of trying to recover the underlying dynamics of a system (the theory) purely from data.

## Miscellaneous Programming

### Slack Bots
[This blog post on building slack bots](https://www.fullstackpython.com/blog/build-first-slack-bot-python.html)

[And this blog post](https://medium.com/@julianmartinez/how-to-write-a-slack-bot-with-python-code-examples-4ed354407b98#.ap4xn8f0h)

[And this wiki](https://botwiki.org/tutorials/slackbots/)

My brother-in-law recently, probably inadvertently, reminded me that technical skills like computer programming don't have to be applied to an immediate world-changing problem to be worthy of time-investment...They can be worthy pursuits simply because they're fun.  The backdrop for this is that I was mentioning that I really wanted to commit some time to learning how to build slack-bots in Python but I hadn't yet figured out a problem in my realm that a slack-bot would really solve (there are probably exceptions but not many scientist I know use slack or other IM clients to chat with colleagues in real time...the pace of scientific discover usually doesn't require immediate co-worker feedback).  My brother-in-law said that at his old job they programmed a slack-bot to listen on all channels and if anyone said, "we're out of beer" the bot responded by ordering more.  

### Web Scraping
[The Beautiful Soup documentation](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)

I'm a big fan of web-scraping and I'm always looking for faster/easier/better ways to do it.  Web scraping is pretty important for my work because a lot of state government agencies aren't sophisticated enough to have well maintained public databases or APIs.  These understaffed agencies usually put a lot of valuable time-series data on the internet in the form of html tables or pdf docs.  I routinely pay undergraduate research assistants decent amounts of money to sit in front of a computer and copy-paste data from public websites to .csv files that I can read into R or Python.

## Books

[Statistics for Spatial Data](http://www.wiley.com/WileyCDA/WileyTitle/productCd-1119114616.html), Chapter 2: Geostatistics
