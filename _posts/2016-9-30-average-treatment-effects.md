---
layout: post
title: "Average Treatment Effect Examples"

---

# Introduction

This workbook present a few simple ways to evaluate average treatment effects when some covariates are correlated with both the assignment to treatment and the outcome variable.  This is sometimes refered to as sample selection bias or the self selection problem.  Let's consider the example from Matai Cattaneo's 2010 *Journal of Econometrics* study of the effect of smoking during pregnancy on babys' birthweights.  The precise empirical problem posed here is that women choose whether or not to smoke, so assignment into treatment and control groups may be non-random.  Moreover, the probability of receiving the treatment (smoke v. not smoke) may be correlated with other variables in the model (like age) which are also correlated with the outcome.  This is sometimes refered to as a confounding variables problem.  

In order to conduct causal inference on the impact of smoking on birthweight we need to estimate the unconditional mean for each group (treatment and control).  What we observe is the outcome conditional on receiving the treatment or not.  In expirimental studies assignment into treatment-control groups is random and therefore uncorrelated with the outcome.  In this case, mean outcomes conditional on the treatment estimate the unconditional mean of interest.  In observational studies, if the assignment to treatment-control groups is non-random we need to model this assignment (called the treatment model).  If our treatment model is any good than the assignment to treatment conditional on covariates can be considered random and estimates of the unconditional group means can be obtained.

## Outline

This workbook illustrates a few approaches for estimating average treatment effects using observational data when self selection into the treatment grouping is a concern:

1. The potential outcome mean estimator (POM)
2. Comparison of weighted group means using inverse probability weights
3. Regression adjustment using propensity scores

## Global Options and Data

```R
#load some libraries
library(ggplot2)
library(dplyr)
```

The data we will use is from from Matai Cattaneo's 2010 *Journal of Econometrics* study.  These data contain, among other things, measurements on birthweight of 4642 newborn babies, maritial status of the mother, age of the mother, and smoking status (smoked v. did not smoke during pregnancy).

Let's get a quick look at the data:

```R
df <- tbl_df(read.csv('data/cattaneo2.csv'))
print.data.frame(df[1:5,])

```

## A Motivating Example: Basic Sample Selection Bias Illustration

The estimation of average treatment effects is often complicated by the issue of non-random assignment to treatment and control groups.  Here we illustrate this problem using some simulated data on baby's birthweight and mother's age and smoker/non-smoker status.  In this case, older mothers are more likely to smoke (read: more likely to receive the treatment) but older mothers also tend to give birth to heavier babies.

```R
#generate some data to illustrate the potential observed mean, probability of being a 
# smoker increases with age
age=rnorm(100,30,5)
#make the probability that a mother smokes increase with age
psmoke <-  pnorm((age-mean(age))/sd(age)) 
smoke <- rbinom(100,1,psmoke) 
#create a dataframe where birthweight is an increasing function of mother's age and smoking status 
z <- data.frame(age=age,smoke=smoke,bw=3000+(5*age)+(25*smoke) + rnorm(100,100,25))
#plot birthweight for smokers and non-smokers 
ggplot(z,aes(x=age,y=bw,color=factor(smoke))) + geom_point() + geom_smooth(method='lm') + ylab("birthweight") + 
  xlab("mother's age") + scale_color_discrete(name="Smoker") + theme_bw()

```

If we are interested in an unbiased estimate of the impact of smoking on birthweight the plot above suggests some issues...namely that we don't have very many smokers on the low end of the age distribution and, relatedly, we don't have many non-smokers in the high end of the age distribution.
