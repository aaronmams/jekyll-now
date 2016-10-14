---
layout: post
title: "State Space Models with Seasonal Data"
---

A little while back [I wrote a few posts on time-series methods for seasonal data](https://thesamuelsoncondition.com/2016/02/19/time-series-iv-markov-regime-switching-models/).  
While I mentioned state space models as an option for modeling seasonal data, I didn't really provide much meat there.  In this post I'm going to give a few examples applying state space models (using the 'KFAS' package in R) to seasonal data.  

Much has been written about state-space modeling and most of it by people with substantially more expertise than I have.  I'm not going to try and compete with the experts on statistical depth....my focus here will be on providing a few reproducible examples you can use to get the feel of how state-space models with the KFAS package behave.

For more depth on state space models I recommend (in order):

* [Durbin and Koopmans, 2001](http://www.ssfpack.com/DKbook.html)
* [Harvey, 1989](https://www.amazon.com/Forecasting-Structural-Models-Kalman-Filter/dp/0521405734)

# Quick State-Space Introduction

As an economist I favor basic illustration of state space models in the context of time-varying regression parameters.  Consider the linear model $y=Z'\alpha$, where $\alpha$ is a vector of coefficients, $Z$ a data matrix, and $y$ a vector of dependent variables.  

If we want to allow the marginal impact of the exogenous variables to evolve over time we might write the system as:

$$y_{t}=Z_{t} \alpha_{t} + \epsilon_{t}$$

$$\alpha_{t+1}=T_{t}\alpha_{t} + R_{t}\eta_{t}$$

The state-space framework is attractive to many researchers because it results in a pretty straightforward likelihood function which can be estimated using familiar methods.


# A Seasonal State Space Model in R

Here we will use some non-confidential data I have on the number of fishing trips taken in the West Coast Groundfish Fishery over time.  The data are monthly from 1994 - 2014 and the variable 'trips' is a count of the total number of fishing trips taken in that month.  The practical background here is that there have been multiple policy changes over time and I am interested in whether any of these regulatory events have altered the seasonal pattern of fishing activity. 

```R
library(Quandl)
library(KFAS)
library(dplyr)
library(ggplot2)
library(lubridate)
library("ggfortify")
library(xts)

#Read in the groundfish effort data------------------------------
df.monthly <- readRDS('data/gf_monthly.RDA')
#----------------------------------------------------------------

```

Next, we'll do some simple processing and plot the series.  I'm cheating a bit here but I know there is a pretty consistent summer peak in these data....In the plot below I mark the 'July' observations with a black dot and other months are red.  

```R
#create a univariate ts object---------------------------------
trips <- df.monthly$ntrips
#--------------------------------------------------------------

#do some other convenience operations---------------------------
df.monthly$date <- as.Date(paste(df.monthly$year,"-",df.monthly$month,"-","01",sep=""),format="%Y-%m-%d")
dates <- df.monthly$date
trips <- xts(trips,order.by=dates)

df.monthly <- df.monthly %>% mutate(july=ifelse(month==7,1,0))
#-----------------------------------------------------------------

#-------------------------------------------------------------------
# a few quick illustative plots
ggplot(df.monthly,aes(x=date,y=ntrips)) + geom_line() + 
  geom_point(aes(color=factor(july))) +  
  scale_color_manual(values=c('red','black')) +
  theme_bw() +
  theme(legend.position='none') 
#---------------------------------------------------------------------

```

![POM_example](/images/north_trips.png)

Note that from 1994 to 2010 we get a consistent seasonal peak in July (with some noise as, in some years the peak is June and others it is August).  Also note that from 2011 to 2014 we get a very consistent August peak.  This is important to me because in Jan. 2011 the West Coast Groundfish Fishery transitioned to a new management model.  An open question, that part of my research will hopefully inform, is whether or not this management transition altred seasonality in the fishery.

Next, I'm going to fit a state-space model to these seasonal data.  First, let's get a little more specific about our seasonal state-space model:

## Seasonal State-Space Model in R and 'KFAS'

The first thing to note here is that the basic presentation of the standard structural time-series decomposition is to express a time-series of observations $y_{t}$ as a function of a level, seasonal, and cyclical component:

$$y_{t}=\mu_{t} + \gamma_{t} + c_{t}$$

where $\mu_{t}$ is the level, $\gamma_{t}$ is the seasonal, and $c_{t}$ is the cyclical component.

Consider the familiar seasonal dummy variable regression model with monthly observations:

$$y_{t}=\alpha + \sum_{i=1}^{11}D_{i}\beta_{i}$$ 

where $D_{i}$ equal 1 if observation $t$ is in month $i$ and 0 otherwise.

If we were to re-write the $\alpha$ term from the original state-space representation as,

$$\alpha_{t} = [\mu_{t},\gamma_{1,t},\gamma_{2,t},...\gamma_{11,t}]$$

then it's pretty easy to see how the basic structural time-series model can be expressed as a state-space model.

Now, we turn our attention to specifying this model in R using 'KFAS':

```R

#-----------------------------------------------------------------------
# A state-space model with a local time-varying level and time varying
# seasonal effects:
tripsmodel<-SSModel(trips ~ SSMtrend(degree = 1, Q=list(matrix(NA))) + 
                       SSMseasonal(period=12, sea.type="dummy", Q = NA), H = NA)
str(tripsmodel)
#-----------------------------------------------------------------------

#------------------------------------------------------------------------
#Estimate the model:
tripsFit<-fitSSM(tripsmodel,inits=c(0.1,0.05, 0.001),method='BFGS')$model

#Recover the smoothed state estimates
tripsSmooth <- KFS(tripsFit,smooth= c('state', 'mean','disturbance'))

#Get smoothed estimates for the level
tripsLevel <-signal(tripsSmooth, states = 'level')
tripsLevel$signal <- xts(tripsLevel$signal, order.by = dates)

#plot level
autoplot(cbind(trips, tripsLevel$signal),facets = FALSE)
#----------------------------------------------------------------------

#----------------------------------------------------------------------
#plot fitted values with raw data
pred <- data.frame(date=dates,kfas=fitted(tripsSmooth),y=as.numeric(trips))

ggplot(pred,aes(x=date,y=y)) + geom_line() + 
  geom_line(aes(x=date,y=kfas),data=pred,color='red') +theme_bw()
#------------------------------------------------------------------------------
```

![kfas fit](/images/trips_tripshat.png)

Next, we extract the smoothed estimates of the state (the monthly seasonal components) and plot them:

```R
#---------------------------------------------------------------------------
#plot the seasonal
trips.sea <- as.numeric(tripsSmooth$alphahat[,2])
plot.df <- data.frame(date=dates,seasonal=trips.sea)
#add a reference point to the seasonal data frame
plot.df <- plot.df %>% mutate(month=month(date),july=ifelse(month==7,1,0))

ggplot(plot.df,aes(x=date,y=seasonal)) + geom_line() + 
  geom_point(aes(color=factor(july))) + scale_color_manual(values=c('red','black'))+
  theme_bw() + 
  theme(legend.position="none")

#----------------------------------------------------------------------------

```

![seasonals](/images/trips_seasonals.png)


