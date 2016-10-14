
---
layout: post
title: "Average Treatment Effect Examples"

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
