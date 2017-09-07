This is a little bit of rehash of [an earlier post on President Cheeto Jesus' economic agenda](https://aaronmams.github.io/Trump'onomics-101/).  The core message is basically the same, but the supporting evidence has been vastly simplified.

# Research Question:

Followers of the Angry Tangerine seem to believe that cutting the corporate tax rate will lead to rainbow stew and blow jobs all around for Blue Collar America.  Is there any sensible basis for this expectation?

# Methods

Plot corporate profits, manufacturing employment, and wages in manufacturing together and look at them.  If we lower the corporate tax rate we expect corporate profits after tax to rise.  By plotting corporate profits after tax over time along with manufacturing employment and earnings we can visualize whether historical increases in corporate profits were positively correlated with increases in either manufacturing employment or earnings. 

Basically, we're asking, when corporations make more money do they hire more blue collar workers or do they pay existing workers more?

# Innovation over my previous post

Rather than monkey around with downloading data from the [Federal Reserve Bank of Saint Louis's FRED database](https://fred.stlouisfed.org/) to a bunch of .csv files and loading those into R, I'm going to use the *Quantmod* package in R to pull data series directly from FRED.

# Results

```R
library(quantmod)
library(dplyr)
library(lubridate)


#nominal hourly earnings in manufacturing (monthly)
getSymbols("CES3000000008",src='FRED')

#GDP deflator
getSymbols("GDPDEF",src='FRED')

#----------------------------------------------------
#real hourly earnings in manufacturing

man.wage <- tbl_df(data.frame(date = index(CES3000000008), 
           nom.wage=CES3000000008, row.names=NULL)) %>%
            mutate(w.man=CES3000000008,year=year(date),month=month(date)) %>%
            group_by(year) %>% 
            summarise(w.man=mean(w.man)) 

deflator <- tbl_df(data.frame(date=index(GDPDEF),def=GDPDEF)) %>%
             mutate(year=year(date),month=month(date)) %>%
             group_by(year) %>% 
             summarise(def=mean(GDPDEF)) %>%
             inner_join(man.wage,by=c('year'))
  
base <- deflator$def[which(deflator$year == 2009)]

deflator <- deflator %>% mutate(real.wage=w.man/(def/base))
(log(deflator$real.wage[deflator$year==2016])-log(deflator$real.wage[deflator$year==2000]))/16

manwage.plot <- ggplot(subset(deflator,year>1960),aes(x=year,y=real.wage)) + geom_line() +
  theme_bw() + ylab("real 2009 $s/hr") + ggtitle("Real wage in manufacturing") + 
  scale_x_continuous(breaks=c(1960,1970,1980,1990,2000,2010,2016))


#----------------------------------------------------


#-----------------------------------------------------
getSymbols("MANEMP",src='FRED')

manemp <- tbl_df(data.frame(date=index(MANEMP),manemp=MANEMP)) %>% 
           mutate(year=year(date),month=month(date)) %>%
           group_by(year) %>% summarise(manemp=mean(MANEMP))

manemp.plot <- ggplot(subset(manemp,year>1960),aes(x=year,y=manemp)) + geom_line() + 
  theme_bw() + ylab('1,000s of workers') + ggtitle('Employment in manufacturing') +
  scale_x_continuous(breaks=c(1960,1970,1980,1990,2000,2010,2016))


#-------------------------------------------------------


#-------------------------------------------------------
#corporate profits

getSymbols("CP",src='FRED')
cp <- tbl_df(data.frame(date=index(CP),cp=CP)) %>%
      mutate(year=year(date),month=month(date)) %>%
      filter(year < 2017) %>%
      group_by(year) %>% summarise(cp=sum(CP))

cp.plot <- ggplot(subset(cp,year>1960),aes(x=year,y=cp)) + geom_line() + 
  theme_bw() + ylab('$ billions') + ggtitle('Corporate profits after tax') + 
  scale_x_continuous(breaks=c(1960,1970,1980,1990,2000,2010,2016))
(log(cp$cp[cp$year==2016])-log(cp$cp[cp$year==2000]))/16
#-------------------------------------------------------


#------------------------------------------------------
require(gridExtra)

grid.arrange(cp.plot, manemp.plot, manwage.plot, ncol=1)

```

![three plots](/images/threeplots.png)
