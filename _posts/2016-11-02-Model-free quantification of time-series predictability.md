---
layout: post
title: "Model-free quantification of time-series predictability"
---

My lab has a small 'quantitative methods' reading group that meets every Thursday.  The idea is to focus on emerging topics in math/statistics/estimation/modeling/etc that might be applicable across disciplines.  Since I work primarily with physical and biological scientists, the papers we read tend to have an ecological bent (Journal of Statistical Physics, Applied Ecology, and Ecology Letters tend to be rich sources for this reading group).  Because of this bent, and the fact that I'm around Ecologists/Biologist/Hydrologists/Geneticists all day (and seldom around Economists), I usually use my blogging time for more pure 'Econ' related pursuits.

However, yesterday's paper kind of hit on the time-series theme that I've been blogging about lately so I thought it might be worth reviewing.  

The paper:

[Model-free quantification of time-series predictability](https://arxiv.org/abs/1404.6823)

A supporting reference:

[Practical consideration of permutation entropy: A tutorial review](http://link.springer.com/article/10.1140/epjst/e2013-01862-7)


## The General Idea

This paper focuses on the relationship between complexity of a time-series and predictability.  My interpretation of what the authors are focusing on is the following: can we measure how much structure exists in a particular time-series that may be exploited in order to make good prediction (if we had the correct model)? 

An aside: I found the paper philosophically interesting because it was really trying to operate in the 'model free' environment.  That is, I read a lot of papers that explore strengths and weakness of different models leveraged for different purposes.  This paper was less interested in identifying the 'right' model and more interested in the question of, "can even the best model accurately predict values in a time-series?"  This appears to be a more active area of research than I realized.  I thought this question was quite novel, turns out it's been under investigation for some time.

## The Set Up
The paper proceeded according to the following basic breakdown:

1. generate several different time-series with different complexity properties
2. Use 4 broad classes of models to try and predict out-of-sample observations of each series.  
3. Calculate the Mean Absolute Scaled Prediction Error for each model applied to each time-series.  MASE is defined as:

$$ MASE=\sum_{j=n+1}^{k+n+1}\frac{|p_{j}-c_{j}|}{\frac{k}{n-1}\sum_{i=2}^{n}|x_{i}-x{i-1}|} $$

4. Quantify the 'complexity' of each time-series.  Philosophically, the authors describe complexity as a function of redundancy, practically, the authors argue that *weighted permutation entropy* is an effective way to measure redundancy.
5. Evaluate the relationship between prediction error and 'complexity' for each model and each time-series.

The four broad classes of model that were used from Step 2 above were:

* naive: the naive model basically says the best predictor of $x_{t}$ given $x_{1},...x_{t-1}$ is the average
* random walk: a model that says the best prediction of $x_{t}$ is $x_{t-1}$
* ARIMA: the auto.arima procedure in R was used to determine the best ARIMA order to predict each time-series
* The Lorenz method of analogues....I won't pretend I know what this is.  I don't.


## The Metrics

One of the big things I got out of this paper was the metric of Permutation Entropy for measuring the 'complexity' of a time-series.  Permutation Entropy is not a new concept/measurement but it was new to me..although saying I was unfamiliar with an arbitrary entropy measure isn't saying much of an consequence.  For whatever reason, I don't use entropy measures very often in my work.

## A Quick Expiriment

I haven't had time to code up my own permutation entropy measure yet...but, as luck would have it, an R package to do it already exists...it's called [statcomp](https://cran.r-project.org/web/packages/statcomp/statcomp.pdf).  For those of you firmly in the 'roll your own camp' the supporting reference I provided above contains some pseudo-code that may be of interest:

### Pseudo Code for calculating permutation entropy:

This pertains to a specialized case.  For the generalized process please refer to the paper.

Consider the time-series x=[6,9,11,12,8,13,5] which has N=7. 

1. The permutation order is set to n=3 and so n! is 6. The permutation possibilities $\pi$ are $\pi_{1}=1,2,3$, $\pi_{2}=1,3,2$, $\pi_{3}=2,1,3$, $\pi_{4}=2,3,1$, $\pi_{5}=3,1,2$, $\pi_{6}=3,2,1$.
2. Initialize i=1 and $z_{j=1,...,6}=0$
3. The rank sequence of the selected values 6,9,11 is 1,2,3.
4. It is equal to $\pi_{1}$, therefore $z_{1}$ is increased to 1.
5. Because $i<7-3$, the next values 9,11,12 are selected which have the rank sequence 1,2,3.
4. It is again equal to $\pi_{1}$, therefore $z_{1}$ is increased to 2.  The loop between Step 3 and Step 5 is then passed through three more times, which leads in the end to $z_{1}=2, z_{2}=0, z_{3}=1, z_{4}=2, z_{5}=0, z_{6}=0$.
6. The values of the counters are divided by *sum=5* which leads to $p_{1}$=2/5, $p_{2}$=0,$p_{3}$=1/5, $p_{4}$=2/5, $p_{5}$=0, $p_{6}$=0.
7.  On the basis of non-zero $p_{j}$, the permutation entropy of order 3 is $H_{3}=(2/5ln(2/5)+1/5ln(1/5)+2/5ln(2/5))$.

### Get permutation entropy for a "predictable" series versus a "complicated" one

```{r}
library(statcomp)

# a really simple series
t <- c(1:100)
xt <- cos(t)
plot(xt,type='l')

#get the pattern distribution and the permutation entropy for this simple series
opd = ordinal_pattern_distribution(x = xt, ndemb = 6)
permutation_entropy(opd)

[1] 0.4202418

# a complicated one...random walk.
M=100;
yt = rnorm(M);
plot(cumsum(yt), type='l')

#get the pattern distribution and the permutation entropy for the more complicated series
opd = ordinal_pattern_distribution(x = yt, ndemb = 6)
permutation_entropy(opd)
[1] 0.6810675
```
Clearly, I'm still pretty green in this area but it is encouraging that the 'simple' time-series ($x_{t}=cos(t)$) has low permutation entropy, indicating less relative complexity than the random walk series.
