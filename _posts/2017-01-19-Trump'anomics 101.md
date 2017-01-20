

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

2.[Total monthly employment in manufacturing for production and nonsupervisory positions for the U.S. in 1,000s of jobs](https://fred.stlouisfed.org/series/MANEMP) is also from FRED.

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
