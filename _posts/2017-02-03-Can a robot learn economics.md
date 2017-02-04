
I spent a non-trivial amount of time this week trying to pick apart the code in R's [rgp](https://cran.r-project.org/web/packages/rgp/index.html) package and Matlab's [GP tips](https://www.mathworks.com/matlabcentral/fileexchange/47197-genetic-programming-matlab-toolbox?requestedDomain=www.mathworks.com) to see if I could modify it to do coupled dynamical systems...I have no notable progress to report on this front.

I do have something a little less cool but a little more digestable ready to go:

[JR Koza](www.genetic-programming.com/ISFB.ps) has a pretty nice vignette/working paper/something on using genetic programming to discover a fundamental macro economic relationship.  I played around with a lot of population dynamics data on predator-prey systems this week and didn't have much success 'discovering' well-established results from theoretical ecology....this problem looked a little simplier so I figured I'd try to replicate what Koza did...with one notable difference of course: Koza uses LISP (sidenote: Fuck that!) and I'm just goign to use R's black box.

# Set Up

Koza's application uses the established relationship between price levels, money supply, interest rates, and production in an economy:

$$P=\frac{M*V}{Q}$$

The data used are the following:

1. Production (Q) is measured with GNP82: the annual rate for the United States Gross National
Product in billions of 1982 dollars.

2. Prices (P) are measured with the Gross National Product Deflator (normalized to 1.0 for 1982).

3. Money Supply is measured with the M2 series: monthly value of the seasonally adjusted money stock M2 in billions of dollars, averaged for each quarter.

4. Interest rates approximate money velocity and are measured with the FYGM3 series: monthly interest rate yields of 3-month Treasury bills, averaged for each quarter.

I pulled these series from the Federal Reserve Bank of St. Louis.  You can get them from the .csv files in [the github repository](https://github.com/aaronmams/gp/tree/master/data).

## A First Pass

```R
library(dplyr)
library(ggplot2)
library(rgp)

####################################################################################
####################################################################################
####################################################################################
####################################################################################
####################################################################################
####################################################################################
####################################################################################
#let's try to replicate the Koza application:

#here we will be trying to recover the function from MacroEcon:
# P=MV/Q

#we want quarterly data from 1959:1 to 1988:4
#-------------------------------------------------------------------
#read data

#GNP:
gnp <- tbl_df(read.csv("/Users/aaronmamula/Documents/R projects/GP/data/GNP82.txt")) %>%
        mutate(DATE=as.Date(DATE,format='%Y-%m-%d')) %>%   
        filter(DATE>='1959-01-01')

#GDP Deflator
gd <- tbl_df(read.csv("/Users/aaronmamula/Documents/R projects/GP/data/GNPDEF.csv"))
new.dates <- strsplit(as.character(gd$DATE[1:88]),"/")

early.dates <- as.Date(unlist(lapply(new.dates,
                                     function(x)return(paste(x[1],"-",x[2],"-",paste("19",x[3],sep=""),sep="")))),
                       format="%m-%d-%Y")
late.dates <- as.Date(gd$DATE[89:279],format="%m/%d/%y")

gd$date <- c(early.dates,late.dates)

#Money Supply
M2 <- tbl_df(read.csv("/Users/aaronmamula/Documents/R projects/GP/data/M2SL.csv")) 

new.dates <- strsplit(as.character(M2$DATE[1:120]),"/")
early.dates <- as.Date(unlist(lapply(new.dates,
                                     function(x)return(paste(x[1],"-",x[2],"-",paste("19",x[3],sep=""),sep="")))),
                       format="%m-%d-%Y")
late.dates <- as.Date(M2$DATE[121:696],format="%m/%d/%y")

M2$date <- c(early.dates,late.dates)

#average money stock to quarterly values
M2 <- M2 %>% mutate(year=year(date),month=month(date),
                    quarter=ifelse(month %in% c(1,2,3),1,
                                   ifelse(month %in% c(4,5,6),2,
                                          ifelse(month %in% c(7,8,9),3,4)))) %>%
     group_by(year,quarter) %>%
     summarise(M2SL=mean(M2SL,na.rm=T)) 

#fix quarterly dates
M2 <- M2 %>% mutate(month=ifelse(quarter==1,4,
                                ifelse(quarter==2,7,
                                       ifelse(quarter==3,10,1)))) %>%
            mutate(year.new=ifelse(quarter==4,year+1,year)) %>%
            mutate(date=as.Date(paste(year.new,"-",month,"-","01",sep=""),format="%Y-%m-%d"))

#T-bill
tbill <- tbl_df(read.csv("/Users/aaronmamula/Documents/R projects/GP/data/TB3MS.csv")) 

new.dates <- strsplit(as.character(tbill$DATE[1:420]),"/")
early.dates <- as.Date(unlist(lapply(new.dates,
                                     function(x)return(paste(x[1],"-",x[2],"-",paste("19",x[3],sep=""),sep="")))),
                       format="%m-%d-%Y")
late.dates <- as.Date(tbill$DATE[421:997],format="%m/%d/%y")

tbill$date <- c(early.dates,late.dates)

#roll up to quarterly
tbill <- tbill %>% mutate(year=year(date),month=month(date),
                    quarter=ifelse(month %in% c(1,2,3),1,
                                   ifelse(month %in% c(4,5,6),2,
                                          ifelse(month %in% c(7,8,9),3,4)))) %>%
  group_by(year,quarter) %>%
  summarise(TB3MS=mean(TB3MS,na.rm=T)) 


#fix quarterly dates
tbill <- tbill %>% mutate(month=ifelse(quarter==1,4,
                                 ifelse(quarter==2,7,
                                        ifelse(quarter==3,10,1)))) %>%
  mutate(year.new=ifelse(quarter==4,year+1,year)) %>%
  mutate(date=as.Date(paste(year.new,"-",month,"-","01",sep=""),format="%Y-%m-%d"))
#-------------------------------------------------------------------


#fix GDP Deflator...in the Koza study it was pegged to 1982 = 1 but
# we'll just use whatever base year it's already in
names(gnp) <- c('date','GNP')
gnp <- gnp %>% inner_join(gd,by=c('date')) %>% mutate(gnp.real=GNP/(GNPDEF/100))

#####################################################################################
#####################################################################################
#####################################################################################
#####################################################################################
#####################################################################################
#####################################################################################

#####################################################################################
#####################################################################################

#in Koza's analysis the price level can be expressed as
# GD (t)= (M2(t)*1.6527)/GNP82(t)


#our first test will be to simulate some data from this type of system 
# and see if the GP can recover something similar

#first make everything the same size
gd <- gd %>% filter(date>'1959-01-01' & date<'1988-10-01')
M2 <- M2 %>% filter(date>'1959-01-01' & date<'1988-10-01')
gnp <- gnp %>% filter(date>'1959-01-01' & date<'1988-10-01')
tbill <- tbill %>% filter(date>'1959-01-01' & date<'1988-10-01')

eps <- rnorm(118)
ysim <- (M2$M2SL * (1.65+eps))/gnp$gnp.real

plot(M2$date,ysim,type="l")


#Basic building blocks
functionSet1 <- functionSet("+","-","*","/")
inputVariableSet1 <- inputVariableSet("m","g")
constantFactorySet1 <- constantFactorySet(function() rnorm(1))

#Defining the 'fitness' function

fitnessFunction1 <- function(f){
  fitness <- rmse(Vectorize(f)(M2$M2SL,gnp$gnp.real),ysim)
  return((if(is.na(fitness)) Inf else fitness))
}

#performing the GP run
#set.seed(1)
gpResults1 <- geneticProgramming(functionSet=functionSet1,
                                 inputVariables=inputVariableSet1,
                                 constantSet=constantFactorySet1,
                                 fitnessFunction=fitnessFunction1,
                                 extinctionPrevention=TRUE,
                                 populationSize=300,
                                 stopCondition=makeTimeStopCondition(2*60))


> gpResults1$elite
[[1]]
function (m, g) 
1.73005737513033 * (m + ((m - (g - (m + m + m - m)))/0.380410797131047 - 
    g)/m)/g

[[2]]
function (m, g) 
1.73005737513033 * (m + ((m - (g - (m + m + m - m)))/0.380410797131047 - 
    g)/m)/g

[[3]]
function (m, g) 
1.73005737513033 * (m + ((m - (g - (m + m + m - m)))/0.380410797131047 - 
    g)/m)/g
    
    
[[29]]
function (m, g) 
1.73005737513033 * (m + -21.4294976588751)/g

[[30]]
function (m, g) 
1.73005737513033 * (m + -21.4294976588751)/g    
```

A few quick observations from the first run:

First, the top programs recovered don't seem to have any strong visual similarities with the identity generating the data.  The top function simplifies to,

$$ 1.7(m + (\frac{-m-g}{0.38-g}\frac{1}{m})/g$$

where m is the M2 money supply and g is the GNP series (Q).

Second, if we look a little further down on the list of elite programs returned by the first run we do see somethings that look vaguely similar to (at least) the shape we are looking for:

$$ 1.7*\frac{m-21.4}{g}$$

Third, just to add to the optimism from 2: I ran this GP 100 times with a fixed population size of 100 individuals imposing a maximum run time of 2 minutes.  In about 90% of runs the best fit programs was some form of:

$$ f(m,g) = \frac{\alpha*m}{\beta*g} $$

Furthermore, in over 60% of those runs the function,

$$f(m,g)=\frac{m*1.7}{g}$$

was on the Pareto Front...i.e. a non-dominated function.

## Some Wrap-up Observations

1. It is pretty common in the literature on symbolic regression and genetic programming to fixate on the Pareto Front.  This, for economists, is much like an indifference curve: it maps out the programs the tradeoff between program complexity and program fitness.  I tried to use these plots to sort through my results but the R implementation of GP uses a multi-objective search hueristic by default so, as far as I can tell, it incorporates a pretty decent complexity penalty.  As a result, when I plot terminal program population in complexity-fitness space, it's pretty lumpy.

2. R's GP implementation has an option "extinctionPrevention." According the documentation this should try to remove duplicate programs from the population...however, I seem to to keep getting a terminal population at the end of each run that is basically just a bunch of copies of 2 or 3 really fit functions.  Not sure why this is...gonna look into it next week.
