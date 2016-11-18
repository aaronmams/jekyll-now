---
layout: post
title: "Broken Windows, Carrots, Sticks, and Roe v Wade"
---

In 1982 [Kelling and Wilson](http://www.theatlantic.com/magazine/archive/1982/03/broken-windows/304465/) published their article in *The Atlantic Monthly* titled: "Broken Windows: The Police and Neighborhood Safety." Perhaps because I have followed the academic and public policy discussions surrounding "Broken Windows" policing, I was kind of surprised to hear a segment on NPR today that made it clear to me that there are still a lot of people who think the "Broken Windows" metaphore is associated with time-tested successful policing strategies.  I guess I thought everyone basically knew that policing based on the "Broken Windows" hypothesis has been empirically shown to have extrememly questionable/uncertain benefits.  

A Quick Bit of Nomenclature and History:
* In 1969 a Stanford psychologist, Phillip Zimbardo, arranged to have two cars without license plates abandoned on the street.  One in Palo Alto, CA and another in The Bronx, NY.  He observed that the car abandoned in the Bronx was basically ripped apart and sold for parts within 10 minutes of being abandoned.  The car in Palo Alto sat untouched for about a week until Zimbardo himself broke a window.  After that, the car was far game and got ransacked pretty quickly.  He took this as evidence that disorder invites disorder.  
* "Broken Windows" is a theory of policing that seems to have originated with Kelling and Wilson - and has its intellectual roots in the Zimbardo experiment - that says that policing low level offense vigorously reduced more serious crime..."Broken Windows" says if you turn a blind eye to vandalism, graffiti, and various other low level offenses you create an environment that invites more serious crime but if you prosecute these low level offense vigorously you can keep serious crime at bay.  
* In 1990 Bill Bratton was promoted to Chief of New York City's Transit Authority. He implemented what is thought to be one of the earliest and most significant policing strategies based on the "Broken Windows" hypothesis.  He energetically prosecuted fare-dodging, graffitti, and other low level public transit related offenses.  In 1993 when Rudy Guiliani was elected Mayor, Bill Bratton become the Chief of Police and brought his "Broken Windows" inspired policing strategy to the NYPD.  The experience of New York City in the 1980s and 1990s is often used in empirical evaluations of the efficacy of "Broken Windows" policing because of these events.

## Outline

As both of my regular readers know, I like to provide the goals and highlights of every post up-front...that way nobody has to get halfway through a few thousand words before realize that they are not at interested in the topic/analysis at hand.

**Here's What I Plan to Do**
* I've scanned a large swarth of literature that utilizes data to test the efficacy of "Broken Windows" policing.  I've also studied in-depth a few papers in this literature that I consider to be particularly reputable (either because they appear in quality journals or because I scanned the methods sections and found them to be reasonably rigorous).  I'm going to provide a review of these resources viz-a-viz i) how the analysis is conducted, ii) what data sources are leveraged, and iii) what they say positive or negative about the efficacy of policing strategies based on the "Broken Windows" hypothesis.
* For most of the literature I'm going to review here I've tried to ascertain whether the data used is readily available and whether quick replication is possible.  In most cases it is not...but there are some easy to access data sources on historical crime rates.  In the 2nd part of this post I provide some code for accessing and analyzing publicly available data on crime.

**Here is the 10 cent version of what this post will show**

First: what is my the verdict on the "Broken Windows" hypothesis?

1. I think it was a good idea advanced by two well-meaning researchers looking to solve an important problem (how to keep communities safe with limited law enforement resources)

2. Despite being a good idea at the time it was proposed, I believe that a proponderance of the empirical evidence has shown highly uncertain benefits to policing strategies based on the "Broken Windows" philosophy.

3. The metaphore of "Broken Windows" is really powerful in its simplicity.  This is probably why the idea has persisted in the public conciousness despite the fact that data has shown it to be of dubious quality.  I'm generally inclined to forgive researchers who's ideas get hi-jacked to satisfy and agenda...In this case however, I don't think Kelling and Wilson get off that easy since I've heard interviews where at least one has basically admitted that elementary data analysis can quickly cast shadows on the "Broken Windows" hypothesis but said that didn't change his belief that "Broken Windows" was a solid foundation for policing strategies.  Look man, I'm a researcher and a scientist too.  I have ideas and I try to turn them into something that can be helpful.  Sometimes I'm successful and sometimes my ideas are just wrong.  People who are willing to cling to an idea in the face of overwhelming evidence that the ideas is bunk are called advocates, not scientists.

Second: what are some high points from my lit review/data exploration?

1. I was a little dismayed at how difficult it was to find a proper difference-in-differences study on this topic.  This seems like the perfect set-up for a treatment-control study and couldn't find one of the quality that I had hoped would be out there.  Basically, if we want to know if "Broken Windows" policing works we really need to find (a) a place where it was it was implemented and (b) a place that did not implement "Broken Windows" type policing but that is very much like the place that did. 

2. I was a little dismayed at how difficult it was to (a) find out what data authors were using and where it came from and (b) find actual data sources once I thought I understood what data each author was using.  My dismay may partially explain #1...that is, maybe there are no good D-i-D studies because there aren't good time-series data on crime for any areas other than a small number of big metropolitan areas and state-level aggregates...this is a shame.  With the attention that this topic has gotten I would hope it would be easier to compare outcomes for NY and - for example - surrounding smaller cities that may police differently.

3. I was more than a little dismayed at how easy it was for me to find strong empirical evidence that 'broken windows' is a broken/useless theory and how many people supporting 'broken windows' are still clinging to it's value even in the face of overwhelming empirical evidence that it doesn't work.

4. I've had some harsh words for Steve Levitt and Freakonomics in the past.  I didn't love the Levitt analysis from Freakonomics on abortions and crime.  I thought it was seriously flawed but it does provide a refreshing wake up call on the 'broken windows' discussion.  That is, one of Levitt's explanations for the reduction in the crime rate in the 1990s was the success of Roe v. Wade in 1970.  On it's face that explanation seems far-fetched and much less intuitive than the 'broken windows' explanations.  However, 'broken windows' has been shown to have no more statistical power to explain the drop in crime in NY in the 1990s than Levitt's alternative hypothesis.  To me what that says is that the 'broken windows' crowd has posited an explaination for declining crime rates that is simple and intuitive yet has no more empirical validity than a far-fetched, seemingly rediculous theory about abortion rates.  If it takes more than that the finally put the nail in the 'broken windows' coffin than I really don't understand how people evaluate evidence and form opinions.

## Part I: A Review of Some "Broken Windows" Literature

There are 4-5 papers I found in the scholarly literature that I felt were sufficiently data driven to warrant inclusion in this post (Note: there are tons of papers on this topic in the law literature.  That shit is not really my bag baby...it tends to be really wordy, the papers are really long, and there are no equations.  The no equations thing might be a selling point for some people so don't let me turn you off of the law review lit...but don't expect me to read it.)

### Levitt: Journal of Economic Perspectives

This is probably one of the most compact, concise, and unapologetic rebukes of "Broken Windows" that you'll find.  It was published in the Journal of Economic Perspectives but if you don't have scholarly journal access [a .pdf is available here](http://pricetheory.uchicago.edu/levitt/Papers/LevittUnderstandingWhyCrime2004.pdf)...It's called "Understanding why crime fell in the 1990: four factors that explain the decline and six that do not.  

I won't leave you in suspense.  The 4 factors that matter, according to Steve Levitt are:

1. Increase in the number of police
2. Rising prison population
3. The receeding crack epidemic
4. Legalization of abortion

The 6 factors that had no influence:

1. The strong economy of the 1990
2. Changing demographics
3. Better policing strategies
4. Gun control laws
5. Concile-and-carry laws - I thought this one was interesting.  From Levitt's paper, *The empirical work in support of this hypothesis, however, has proven to be fragile along a number of dimensions (Black and Nagin, 1998; Ludwig, 1998; Duggan, 2001; Ayres and Donohue, 2003). First, allowing concealed weapons should have the greatest impact on crimes that involve face-to-face contact and occur outside the home where the law might affect gun carrying. Robbery is the crime category that most clearly  ts this description, yet Ayres and Donohue (2003) demonstrate that empirically the passage of these laws is, if anything, positively related to the robbery rate. More generally, Duggan (2001) nds that for crimes that appear to decline with the law change, the declines in crime actually predate the passage of the laws, arguing against a causal impact of the law. Finally, when the original Lott and Mustard (1997) data set is extended forward in time to encompass a large number of additional law enactments, the results disappear (Ayres and Donohue, 2003). Ultimately, there appears to be little basis for believing thatconcealed weapons laws have had an appreciable impact on crime.*
6. Increased use of capital punishment



