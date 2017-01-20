

As a human being I think Trump as president is a disaster for our country...as an economist I think it's pretty unlikely that Trump's presidency will be an economic catastrophe (famous last words, I know).  To be clear, I don't think he'll be great for the economy...but the U.S. economy is pretty big and resilient animal.  Pretty much every Republican thought President Obama would be an economic nightmare and yet, 8 years later, we've enjoyed steady job growth, steady growth in corporate profits, and record highs in pretty much all of our domestic equities markets.  

Of course there are some people who didn't get a share of the economic and financial gains of the past 8 years (those 'left behind by the economic recovery')...and there are some people who didn't get as big a share as they thought they were entitled to.  These are the people I'm interested in today.  Just to be clear, there are plenty of people who voted for Trump out of old fashioned racism, misogyny, and xenophobia (I contend this is the lion's share of Trump voters)...but there's not much to analyze viz-a-viz this group's fortunes.  They're a people who don't care much about their own economic circumstances; they're happy to suffer as long as they don't have to watch people they don't like (the gays, the immigrants, the uppity bitches, whatever) succeed.  

I can't do much about the cultural anarchists.  But there's another group (one that although probably not that big in number almost certainly pushed Trump over the edge in terms of EC votes) that probably honestly believes that their lives will, if not get better, then at least not get any worse with President Trump.  

This post is all about what do I think will be the likely impact of a Trump presidency on the people who voted him in? And why do I think it?

To get more specific, I'm starting with the commonly accepted narrative that Trump cleaned house among undereducated, white male voters.  To get even more specific, my thought experiment is aimed at 'estimating' the impact of 3 agenda items Trump has been vocal about:

1. Repatriating overseas corporate profits: a corporate tax holiday
2. 'Bringing manufacturing back'
3. Energy policy aimed at dramatically increasing domestic oil production

In thinking about three initiatives I want to try and discern how they might help Trump voters 'left behind by the economic recovery'...I define this group as:

* white
* male
* no college education
* unemployed or underemployed unskilled labor 
* living in a Rust Belt or Automobile Alley state 

If you want a more vivid picture of whose economic fortunes I'm trying to contemplate just run with whatever comes into your head when I say, "he's one of those 'the robot took my job' guys."  That sounds pejorative but there are over 1 million currently unemployed former manufacturing employees in the Upper Midwest many of whom voted for Trump because they perceived him to be the candidate most likely to improve their economic circumstances.  There are also hundreds of thousands of manufacturing job opening going unfilled right now because of a basic skills gap.  

    
# Corporate Tax Holiday

let's start with repatriating cash because I think that's a relatively easy one.  In my assessment a corporate tax holiday would be really good for 

* Microsoft
* GE
* Apple
* Pfizer

These companies all currently have somewhere between $60 and 100 billion sitting overseas...Trump has suggested levying a one-time corporate income tax of 10% (vs. the normal 35%) on repatriated profits.  

In short, I think repatriating overseas corporate profits is unlikely to have much benefit for unskilled, undereducated labor.  My conclusion comes from the following observations:

## 1. Corporate profits and Manufacturing Employment

Claim:  corporate profits are not tightly coupled to employment in the manufacturing sector.  

Data:

1. [Corporate profits come from FRED](https://fred.stlouisfed.org/series/CP)...specifically https://fred.stlouis.org/series/CP.  These are quarterly before tax nominal profits that I adjusted to real profits using the implicit GDP price deflator (also from FRED). 

2. [Total monthly employment in manufacturing for production and nonsupervisory positions for the U.S. in 1,000s of jobs](https://fred.stlouisfed.org/series/MANEMP) is also from FRED.

3. [Total monthly employment in motor vehicle and motor vehicle parts manufacturing in Michigan in 1,000s of jobs](https://fred.stlouisfed.org/series/SMU26000003133630001) is also from FRED...specifically series SMU26000003133630001

4. [Total monthly employment in motor vehicle and motor vehicle parts manufacturing in Indiana in 1,000s of jobs](https://fred.stlouisfed.org/series/SMU18000003133630001) is from FRED...specifically series SMU18000003133630001

Analysis:

```r
library(lubridate)
library(dplyr)
library(ggplot2)
library(ggthemes)

pi.corp <- read.csv('/Users/aaronmamula/Documents/R projects/macroecon/corporateprofits.csv')
wages <- read.csv('/Users/aaronmamula/Documents/R projects/macroecon/hourlywage_production_nonsup.csv')
names(wages) <- c('observation_date','wages')
def <- read.csv('/Users/aaronmamula/Documents/R projects/macroecon/GDPDEF.csv')
manemp <- read.csv('/Users/aaronmamula/Documents/R projects/macroecon/manemp.csv')
manemp.MI <- read.csv('/Users/aaronmamula/Documents/R projects/macroecon/manufacturing_motorvehicle_MI.csv')
names(manemp.MI) <- c('observation_date','manemp_mv_MI')
manemp.IN <- read.csv('/Users/aaronmamula/Documents/R projects/macroecon/manufacturing_motorvehicle_IN.csv')
names(manemp.IN) <- c('observation_date','manemp_mv_IN')


#wages are monthly so let's take the quarterly average so we can integrate GDP Deflator
# note that quarters are always defined as 1, 4, 7, 10 in FRED data
wages <- tbl_df(wages) %>%  arrange(observation_date) %>% 
          mutate(month=month(observation_date)) %>%
          mutate(q.avg=ifelse(month %in% c(1,4,7,10),(lag(wages,1)+lag(wages,2)+lag(wages,3))*(1/3),0))

manemp <- tbl_df(manemp) %>%  mutate(observation_date=as.Date(observation_date,format="%Y-%m-%d")) %>% 
  arrange(observation_date) %>% 
  mutate(month=month(observation_date)) %>%
  mutate(emp.avg=ifelse(month %in% c(1,4,7,10),(lag(MANEMP,1)+lag(MANEMP,2)+lag(MANEMP,3))*(1/3),0))
  
manemp.MI.q <- tbl_df(manemp.MI) %>%  mutate(observation_date=as.Date(observation_date,format="%Y-%m-%d")) %>% 
  arrange(observation_date) %>% 
  mutate(month=month(observation_date)) %>%
  mutate(emp.avg=ifelse(month %in% c(1,4,7,10),(lag(manemp_mv_MI,1)+lag(manemp_mv_MI,2)+lag(manemp_mv_MI,3))*(1/3),0)) %>%
  filter(emp.avg!=0)

manemp.IN.q <- tbl_df(manemp.IN) %>%  mutate(observation_date=as.Date(observation_date,format="%Y-%m-%d")) %>% 
  arrange(observation_date) %>% 
  mutate(month=month(observation_date)) %>%
  mutate(emp.avg=ifelse(month %in% c(1,4,7,10),(lag(manemp_mv_IN,1)+lag(manemp_mv_IN,2)+lag(manemp_mv_IN,3))*(1/3),0)) %>%
  filter(emp.avg!=0)

#fix first obs
wages$q.avg[which(wages$observation_date=='1980-01-01')] <- 6.82
manemp$emp.avg[which(manemp$observation_date=='1980-01-01')] <- 19282

#keep only the quarterlys
wages <- wages %>% filter(q.avg!=0) %>% mutate(date=as.Date(observation_date,format="%Y-%m-%d"))
manemp <- manemp %>% filter(emp.avg!=0) 

#merge the deflator to get real wages
def <- tbl_df(def) %>% mutate(date=as.Date(observation_date,format="%Y-%m-%d"))

wages <- wages %>% inner_join(def,by=c('date')) %>%
          mutate(pidx=GDPDEF/100,
                 real.wage=q.avg/pidx)
ggplot(wages,aes(x=date,y=real.wage)) + geom_line()

#put corporate profits in real terms
pi.corp <- tbl_df(pi.corp) %>% mutate(date=as.Date(observation_date,format="%Y-%m-%d")) %>%
              inner_join(def,by=c('date')) %>% mutate(pidx=GDPDEF/100,
                                                      pi.real=CP/pidx) %>%
          select(date,CP,pi.real)

ggplot(pi.corp,aes(x=date,y=pi.real)) + geom_line()

#-----------------------------------------------------
#plot growth in corporate profits relative to 
# US manufacturing employment
# Motor Vehicle manufacturing in MI
# Motor Vehicle manufacturing in IN

tmp1 <- pi.corp %>% select(date,pi.real) %>% 
          filter(date >= '1990-04-01') %>% mutate(series='pi corporate')
names(tmp1) <- c('date','value','series')

tmp2 <- manemp %>% select(observation_date,emp.avg) %>%
          filter(observation_date >= '1990-04-01') %>% 
          mutate(series='manemp')
names(tmp2) <- c('date','value','series')

tmp3 <- manemp.MI.q %>% select(observation_date,emp.avg) %>%
  filter(observation_date >= '1990-04-01') %>% 
  mutate(series='manemp.MI')
names(tmp3) <- c('date','value','series')

tmp4 <- manemp.IN.q %>% select(observation_date,emp.avg) %>%
  filter(observation_date >= '1990-04-01') %>% 
  mutate(series='manemp.IN')
names(tmp4) <- c('date','value','series')

plot.tmp <- rbind(tmp1,tmp2,tmp3,tmp4) %>%
            group_by(series) %>%
            arrange(series,date) %>%
            mutate(base=first(value),
                   g=value/base)
  
ggplot(plot.tmp,aes(x=date,y=g,color=series)) + geom_line() + theme_bw() + 
     ylab("value/base") + xlab("")
#-----------------------------------------------------

```
![pi_corporate](/images/corporateprofits_manemp.png)

I don't see much here telling me that corporate profits are really good for unskilled manufacturing workers.

## 2. Corporate Cash and Stock Ownership

I believe it's highly likely that repatriating large sums of cash will have enormous benefits for wealthy people.  This is because when companies (especially publicly traded ones like IBM, Microsoft, Apple, Pfizer, etc) are blessed with cash windfalls they tend to do things distribute those profits to shareholder in the form of dividends and execute stock buybacks which pushes stock prices higher (benefiting shareholder at large but especially wealthy executives that are paid in stock).

Claim: The types of activities I mentioned above are not going to benefit the 'Trump voters' I'm focused on here.  

Evidence:

[This Pew research report from 2013 detailing who has money in the stock market and who doesn't](http://www.pewresearch.org/fact-tank/2013/05/31/stocks-and-the-recovery-majority-of-americans-not-invested-in-the-market/). 

About 23% of Americans with less education than a college degree have money in the stock market.  

I don't want to spend too much time on this because it seems really obvious to me: if Apple, Microsoft, GE, etc. bring billions of dollars home and distribute a bunch of it to share holders it aint' gonna help the people we're talking about here.  It's not. Let's move on.

## 3. Corporate Cash and Investment

The amount of money we are talking about here could also be used by corporations to beef up R&D or to acquire a presence in a hot area.  Some pretty sharp people that I listen to have made the following observations:

1. IBM was pretty late to the enterprise cloud computing party.  Microsoft and Amazon have been killing it in that area lately and, if IBM, suddenly had access to $60 billion (maybe as a result of a tax holiday) they might think pretty seriously about buying their way into the lucrative enterprise cloud computing services game.

2. Gene-editing technologies are pretty much the hottest thing in biotech these days.  Editas Medicine, Intellia Therapeutics, and CRISPR Therapeutics are some of the gene-therapy stocks that a lot of people have their eyes on.  If a company like Pfizer suddenly had 10's of billions of dollars lying around it could buy into the gene-therapy game pretty easily.

I don't have much empirical evidence to offer here...I just can't imagine this type of acquisition really helping out unskilled labor.  To the extent that investment or acquisition by the companies with lots of cash parked overseas results in manufacturing jobs, I'm pretty sure those jobs are going to be on the higher skilled end of the spectrum...like for people who can run 3D printers and know what nanotechnologies are and shit.  Don't worry I'll cover this 'skills gap' argument more in the next section.

# Bringing Manufacturing Back

Trump and Trump supporters have been notably vocal about 'bringing the manufacturing sector back.'  You usually here this policy prescription right after someone bemoans, "We don't make/build stuff anymore."  

First, let's establish the facts:

## Manufacturing employment is down...sort of

Employment in manufacturing (at least in traditionally manufacturing dependent states) is down a lot from the 2000s...but in every state I looked at it has been trending pretty steadily up since 2010.

```r
#-----------------------------------------------------
#plot manufacturing employment in MI, IN, AL

manemp.IN.all <- tbl_df(read.csv('/Users/aaronmamula/Documents/R projects/macroecon/manemp_all_IN.csv')) %>%
                    mutate(state='IN') %>%
                    mutate(date=as.Date(observation_date,format="%m/%d/%y")) %>%
                    select(date,INMFG,state)
manemp.MI.all <- tbl_df(read.csv('/Users/aaronmamula/Documents/R projects/macroecon/manemp_all_MI.csv')) %>%
                    mutate(state='MI')
manemp.WI.all <- tbl_df(read.csv('/Users/aaronmamula/Documents/R projects/macroecon/manemp_WI_all.csv')) %>%
  mutate(state='WI')

manemp.AL.all <- tbl_df(read.csv('/Users/aaronmamula/Documents/R projects/macroecon/manemp_all_AL.csv')) %>%
                    mutate(state='AL')
manemp.TN.all <- tbl_df(read.csv('/Users/aaronmamula/Documents/R projects/macroecon/manemp_TN_all.csv')) %>%
  mutate(state='TN')

names(manemp.IN.all) <- c('date','emp','state')
names(manemp.MI.all) <- c('date','emp','state')
names(manemp.AL.all) <- c('date','emp','state')
names(manemp.WI.all) <- c('date','emp','state')
names(manemp.TN.all) <- c('date','emp','state')

#use a moving average to get trend
plot.df <- rbind(manemp.IN.all,manemp.MI.all,manemp.AL.all,manemp.WI.all,manemp.TN.all) %>% 
          group_by(state) %>% arrange(state,date) %>%
          mutate(region=ifelse(state %in% c("MI","IN","WI"),'Rust Belt','South'))

ggplot(plot.df,aes(x=date,emp,color=state)) + geom_line() + facet_wrap(~region) + 
  theme_bw() + xlab("") + ylab("thousands of employees")

#----------------------------------------------------
```

![manemp_states](/images/manemp_states.png)

## Manufacturing pay is up...or not

Data series:

* Average weekly earnings of all manufacturing employees in Wisconsin, dollars per week, not seasonally adjusted: https://fred.stlouisfed.org/series/SMU55000003000000011
* Average weekly earnings of all manufacturing employees in Michingan, dollars per week, not seasonally adjusted: https://fred.stlouisfed.org/series/SMU26000003000000011
* Average weekly earnings of all manufacturing employees in Indiana, dollars per week, not seasonally adjusted: https://fred.stlouisfed.org/series/SMU18000003000000002
* Average weekly earnings of all production and non-supervisory manufacturing employees in the U.S, dollars per week, not seasonally adjusted: https://fred.stlouisfed.org/series/CES3000000030
* CPI for all urban consumers, all goods, index values, 1982-1984=100: https://fred.stlouisfed.org/series/CPIAUCSL 

At the national level real weekly pay in manufacturing is down from the good ol' days but has been steadily improving since 2010...and is currently higher than at any other period in time with the exception of the boom of the 70s and 80s.

```r
#use cpi to convert weekly wage to real terms
wage_week <- tbl_df(wage_week) %>% 
  inner_join(cpi,by=c('date')) %>%
  mutate(real.pay=pay/(cpi/100),
         date=as.Date(date,format="%Y-%m-%d"))

ggplot(wage_week,aes(x=date,y=real.pay)) + geom_line() + theme_bw()

```

![weeklypay_US](/images/weekly_pay_man_US.png)


For some selected bell-weather states real pay seems pretty stagnant (nominal pay is the black line and real pay is the red one):

```r
#-----------------------------------------------------
#avg weekly pay in manufacturing

#by state
pay.MI  <- tbl_df(read.csv('/Users/aaronmamula/Documents/R projects/macroecon/weekly_earnings_MI_man.csv')) %>%
  mutate(state='MI', date = as.Date(observation_date,format="%Y-%m-%d"),
         year=year(date)) %>% select(-observation_date) 
names(pay.MI) <- c('pay','state','date','year')

pay.WI  <- tbl_df(read.csv('/Users/aaronmamula/Documents/R projects/macroecon/weekly_earnings_WI_man.csv')) %>%
  mutate(state='WI', date = as.Date(observation_date,format="%Y-%m-%d"),
         year=year(date)) %>% select(-observation_date) 
names(pay.WI) <- c('pay','state','date','year')

pay.IN  <- tbl_df(read.csv('/Users/aaronmamula/Documents/R projects/macroecon/weekly_earnings_IN_man.csv')) %>%
  mutate(state='IN', date = as.Date(observation_date,format="%Y-%m-%d"),
         year=year(date)) %>% select(-observation_date) 
names(pay.IN) <- c('pay','state','date','year')

pay <- rbind(pay.MI,pay.WI,pay.IN)

#bring in CPI
cpi <- cpi %>% mutate(date=as.Date(date))
pay <- pay %>% left_join(cpi,by=c('date')) %>%
          mutate(real.pay=pay/(cpi/100))

ggplot(pay,aes(x=date,y=pay)) + geom_line() + geom_line(aes(x=date,y=real.pay,color='red')) + 
  facet_wrap(~state) + theme_bw() + guides(color=FALSE) + xlab("") + 
  ylab("weekly pay")

```

![weeklypay_states](/image/weekly_pay_real.png)


## Manufacturing output is waaaaay up

There's no way around this one.  Productivity increases have pushed U.S. manufacturing output to historically high levels.

Data:

1. [FRED, real U.S. manufacturing output, index values, 2009 = 100, not seasonally adjusted](https://fred.stlouisfed.org/series/OUTMS)

![manout](/images/manout.png)


## Discussiong of 'bringing manufacturing back'

Ok, let me try to tie some of these observations together:

Trump wants to "bring manufacturing back" but manufacturing - the activity - never went anywhere.  We are producing more shit than we ever have...we just don't need as many people to do it.  To me it's pretty clear that 'bringing manufacturing back' (to extent it is successful at all) is going to fall into one of two categories:

1.  Continuing the manufacturing resurgence that has been going on since at least 2010, or
2.  Somehow conducting a ressurection of old school labor intensive manufacturing that if friendly to unskilled workers

### Continuing the trend

U.S. manufacturing has evolved to do things: 

1. focus on high margin output (comparative advantage...let low wage countries make low-margin shit and use our relatively high paid workers to make more sophisticated shit).
2. take advantage of new technologies to make more stuff with less money/labor/time/etc.

In my mind, the problem for those 'left behind by the economic recovery' isn't so much that manufacturing is dead...it's that manufacturing now needs skills they don't have.

There is a well documented 'skills gap' in U.S. manufacturing.  In fact, the Manufacturing Institute reported that there are several hundred thousand manufacturing job openings unfilled currently because of a lack of qualified workers. Here are three interesting resources on the current skills gap in manufacturing:

1. [U.S manufacturing has seen some robust job growth recently but most of the jobs are going to skilled college educated workers...not unskilled labor](http://blogs.cisco.com/manufacturing/then-and-now-why-manufacturing-isnt-what-it-used-to-be)

2. [This articles details where the growth in the manufacturing sector has been recently](http://cerasis.com/2015/01/13/manufacturing-technology/)...basically, machines have increased labor productivity immensely and things like nanotechnologies and 3D printers have made it possible for us to manufacture things on a large scale that we couldn't before.

3. [This article echos my ponit that manufacturing employment has increased since 2010....but most of those of jobs have not gone to unskilled labor...they have been tech driven](https://www.technologyreview.com/s/602869/manufacturing-jobs-arent-coming-back/).

4. [The Manufacturing Institute](http://www.themanufacturinginstitute.org/Research/Facts-About-Manufacturing/Workforce-and-Compensation/Workforce-by-Education/Workforce-by-Education.aspx) reports that he percent of jobs in manufacturing held by individuals with a high school education but no college dropped 3.5% between 2000 and 2012.  It also shows that the percent of total jobs in manufacturing held by those with a bachelor's degree increased from 16% in 2000 to 20% in 2012.  Given the current pace of technology I find it likely that the college educated as a share of the manufacturing workplace continued to growth between 2012 and today...I find it hard to believe that this trend is likely to reverse itself.

5. [This article](http://knowledge.wharton.upenn.edu/article/can-trump-anyone-bring-back-american-manufacturing/) points out that Trump can't really bring manufacturing jobs back because there's nowhere to bring them back from.  It claims that 80% of lost U.S. manufacturing jobs weren't lost to China or Mexico, they were lost to automation.

## Bucking the trend

For fun, let's image that somehow Trump (because he's such a righteous deal maker) is able to create tons of jobs for low skilled, undereducated workers.  Suppose he gets Apple to start making iphones in the U.S. or something.  Will that help the Trumpkins?  Probabily....a little.  

It would almost certainly create some jobs for unskilled workers.  However, referring back to chart above of weekly manufacturing pay by state, it's pretty clear that pay is not keeping up with inflation for a lot of existing manufacturing workers. Wages in the sector are already stagnant.  Apple would stand to lose A LOT of money by shifting production to the U.S.  

Given these two facts, it's pretty hard for me to see Apple coming into Ohio or somewhere and just makin' it rain cash.  I find it infinitely more plausable that, unemployment among unskilled workers would probably tick down some but not nearly as much as that group might expect as Apple will turn over every stone looking for places where they can substitute capital/machinery for labor.  See when you're [paying a Chineese dude $1.50/hr](http://www.businessinsider.com/china-labor-watch-apple-iphone-workers-2013-7) you don't have to look to hard for labor saving technologies...when you all of a sudden gotta pay the dude $15/hr you get resourceful real fast.


Let me be clear, I don't think Trump (or anyone else) can actually attract/create the kind of manufacturing jobs that will really benfit unskilled labor.  The reason why is that if a company like Apple has to pay a dude in Indiana $15/hr to put together iphones (and give him benefits) rather than [pay a Chineese dude $1.50/hr](http://www.businessinsider.com/china-labor-watch-apple-iphone-workers-2013-7) the pain is gonig to be felt by a massive number of people...many of whom are some of the wealthy and most powerful people on the planet.

I pointed out above the results of a Pew Research Poll that basically shows poor people and non-college grads don't own stocks.  There is a collolary to that: rich people and college grads basically all own stocks.  And they don't just own stocks, they get paid in stocks.  They have massive sections of their wealth tied up in stocks.  Big companies like Apple, IBM, and GE make up huge weights inside basically every pension fund, retirement fund, mutual fund, and ETF in the U.S.  

Actually, the more fundamental reason come from the Wharton study I linked to above: we can't bring back the kinds of manufacturing jobs friendly to unskilled workers because there aren't any of those jobs left:

* making t-shirts? nope: 3D printers and nano-technology have turned this into a software driven industry
* making cars? there's plenty of evidence that car manufactures will move and alter supply according to their margins.  Ford makes F-150s with backup cameras, park assist, and navigation system in the U.S. because it demands a skilled labor force.  They make Fiestas and shit in Mexico because production isn't as tech driven.  If they have to move production of the Ford Fiesta to Deerborn by order of the Trump, I think it's highly likely they would just stop making it and hire more tech-savy workers to crank out more F-150s and Expenditions.

# Energy Policy

Ok, so the last thing I was thinking about is a little off-the-cuff...I don't think I'll have time to pull much data on this but I'll try to keep the logic as tight as possible and the assumptions as clear as possible so - for all of you dissenters - you can pinpoint whether you disagree because you think my facts are wrong or because you think my induction is off.

Here are my assumptions:

* Trump wants us to pump more oil and gas
* The break even price for U.S. shale producers is approximately normally distributed with mean around $60/barrel West Texas Crude
* Oil trades on a global market subject to global supply and demand forces
* Gas is currently really cheap partly because oil is really cheap: the WTI forward contract is currently trading around $52
