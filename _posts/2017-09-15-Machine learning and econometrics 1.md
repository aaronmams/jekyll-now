
# Introduction

I have a post locked-and-loaded where I conducted my first experiment with [Python's sci-kit learn module](http://scikit-learn.org/stable/).  This module makes it embarrassingly easy to run pretty much any flavor of machine learning model with pretty minimal code.  I used the wrappers from this module to run a suite of classification algorithms

* logistic regression
* linear discriminant analysis
* support vector machine
* k-nearest neighbor
* gaussian naive bayes
* classification and regression trees

on the famous Iris dataset.

I wrote this as a sort of companion to that quick-and-dirty machine learning exploration.

## Some philosophical bullshit

I like all kinds of new shit and I try not to be one of those curmudgeonly academics who are chronically unimpressed with new techniques because they think everything is just some trivial flourish to an old model....but I see a lot of people using machine learning as a tool without considering whether it's the right tool (killing cockroaches with a flame-thrower, pounding nails with a sledgehammer, whatever your favorite metaphor is here).  

I don't really want to get into an arcane discussion of the strengths and weaknesses of all possible configuration of the applied math and statistics machinery.  I do want to point out that, at least for me as a social scientist, when deciding on a machine learning approach versus a competing approach a good starting point is usually deciding whether or not you expect the parameters of a model to yield some sort of insight into the dynamics or operation of the system you are studying.

A useful way, I think, to illustrate this point is to do a little side-by-side of two classification algorithms: logistic regression and linear discriminant analysis.  What I want to highlight here is that both approaches can provide a way to classify unobserved outcomes but only logistic regression allows you also attach some meaning to estimated parameters that might generate some important economic understanding.

## Logit and LDA basics

### Logit crash course 

The logit model defines 

$$P(Y_{i}=y_{i}) = \pi_{i}$$

and models the log odds as a linear function of the predictors:

$$ln \frac{\pi_{i}}{1-\pi_{i}} = x_{i}'\beta$$

For a binary classification problem where $y_{i} = [0,1]$ we can write the crucial logit relationship as,

$$P(y=1) = \frac{e^{x'\beta}}{1+e^{x'\beta}}$$

From here one can:

* take a sample of observed data (observed outputs $y_{i}$ and covariates, $x_{i}$)
* use maximum likelihood to estimate the parameters of the model ($\beta_{i}$) that maximize the likelihood of observing data sample given the structure of the model
* use those estimated parameters to predict/classify an unobserved outcome ($y_{j}$) based on a set of observed covariates ($x_{j}$).

It's nice to make out-of-sample predictions or classifications but, as economists, that often is not all we want to do.  For example, in a credit risk application we may have data on a bunch of loans that were granted, an outcome variable indicating whether the loan was paid back or not, and a bunch of covariates like age of the borrower, annual income of the borrower, etc.

For a new individual applying for a loan with age=A, income=I and a bunch of other stuff, we might want to predict the probability that the individual will pay back a loan.  If we estimate the parameters of the logit model using the observed data, we can use that model to predict whether the new individual will pay back the loan.  

If we work for the bank granting the loans then predicting default risk might be all we are interested in.  But I don't work for a bank.  I'm a researcher and I often care not about the ultimate prediction but rather how the system works.  I care about questions like,

*how much more likely is a a 24 year old to default on a loan than a 25 year old?*

The logit model allows me to evaluate questions like this by calculating **Marginal Effects.**  If we consider a simple model with:

* $y=0$ if individual paid back a loan and 1 if the individual defaulted
* $x_{1}$ is age of borrower and $\beta_{1}$ the coefficient on this variable
* $x_{2}$ is annual income of borrower and $\beta_{2}$ the coefficient on this variable

We can get the maximum likelihood estimates of $\beta_{1}, \beta_{2}$ and use them to calculate the marginal effect of 1 year of age on the predicted probability of defaulting on a loan as follows:

$$P(y=1)=\frac{e^{x'\beta}}{1+e^{x'\beta}}$$

$$\frac{dY}{dx_{1}}=\frac{e^{x'\beta}}{1+e^{x'\beta}}\beta_{1}$$

As a quick technical point, the marginal effect of a particular variable in the logit model depends on the value of other variables.  In practice (if all variables are continuous) it is pretty common to set all covariate values to their means and estimate the marginal effect at the means.  If there are categorical/dummy variables in the model there are some tweaks but they aren't particularly onerous. 


### LDA crash course

The general idea behind LDA is to project a covariate space (think of an example with hundreds of possible predictors) onto a smaller space in a way that achieves maximum separability between classes.  It is similar in theory and execution to logistic regression but differs somewhat in this primary motivation.  

Depending on one's preferred presentation, LDA can be thought of as two sub-processes:

1. project the covariate vector into a lower dimensional subspace
2. apply a decision rule for classifying an unknown observation based on it it's relation to the discriminating function.

For a 2 class model like the credit risk example above we have:

* $y_{i} = [0,1]$
* $x=[x_{1},x_{2}]$

The projection of any observation, $x_{i}$ onto a line in the direction $v$ is,

$$v^{t}x_{i}$$.

Note that the vector $v$ is $[v_{1},v_{2}]$.  It's helpful for me to think of these as weights rather than coefficients.  After projecting each point to the line defined by $v$ 
we can label the mean projected values of each class as $\mu_{1}$ and $\mu_{2}$.

A measure of separation between classes is give by the distance between the means of the projected data

$$|\mu_{1} - \mu_{2}|$$

Fisher's linear discriminant normalizes this distance by a factor proportional to the variance of each class.  Let 

$$z_{i}=v^{t}x_{i}$$

then

$$s_{i}=\sum_{i \in C_{1}}(z_{i}-\mu_{i})^{2}$$

Fisher's linear discriminant projects to a line in the direction $v$ which maximizes the normalized distance between groups:

$$J(v)=\frac{(\mu_{1}-\mu_{2})^{2}}{s_{1}^{2}+s_{2}^{2}}$$

Based on the mechanics of above and some distributional assumptions one can express a linear score function and develop some classification rule for a new observation based on that score.

It is, I think, important to note that the optimized coefficients in this type of model are the,

$$v=[v_{1},v_{2},...]$$

which define how the data in each dimension are projected to the lower dimensional space.  That is, they do not have a straightforward interpretation as marginal effects like the coefficient estimates of a logit model.


# Credit Scoring Data

Ok, enough of the esoteric bullshit.  How bout an example?

I got some credit risk data from lendingclub.com.  I have no idea if it's any good or not but it was free and pretty easy to download.  I got about 40,000 observations on loans from 2007 to 2011 because that's what I could get without opening an account.  This data has an asspile of columns but the ones I'm going to use are:

* interest rate
* annual income of borrower
* debt to income ratio of borrower
* number of delinquent accounts in the past 2 years
* purpose of loan

The response variable is whether or not the individual defaulted on the loan.  It's worth noting here that I coded the variables 1 if the loan was paid back and 0 if the loan defaulted...so the signs on my estimated coefficients are likely to be opposite from what one would expect (since I think it's more conventional to code default as 1).

# Training 

I have 42,506 complete observations in my data set so I'm going to randomly select 8,501 (20%) observations to be my 'validation' set.  The other 80% of the data is going to be used for model fitting.

```R
# the data I use comes from here:
#https://www.lendingclub.com/info/download-data.action

cs <- read.csv('/Users/aaronmamula/Documents/Python Projects/machine_learning/creditscoring.csv')
summary(cs)

#add some dummy variables because I like those better for doing marginal effects
cs <- tbl_df(cs) %>% mutate(paid = ifelse(loan_status %in% c('Fully Paid','Does not meet the credit policy. Status:Fully Paid'),1,0)) %>% 
  select(int_rate,dti,annual_inc,delinq_2yrs,paid,purpose) %>%
  mutate(D_pur_credit=ifelse(purpose=='credit_card',1,0),
         D_pur_car=ifelse(purpose=='car',1,0),
         D_pur_smallbiz=ifelse(purpose=='small_business',1,0),
         D_pur_other=ifelse(purpose=='other',1,0),
         D_pur_wedding=ifelse(purpose=='wedding',1,0),
         D_pur_debt = ifelse(purpose=='debt_consolidation',1,0),
         D_pur_homeimp=ifelse(purpose=='home_improvement',1,0),
         D_pur_majorpurch=ifelse(purpose=='major_purchase',1,0),
         D_pur_medical=ifelse(purpose=='medical',1,0),
         D_pur_moving=ifelse(purpose=='moving',1,0),
         D_pur_vacation=ifelse(purpose=='vacation',1,0),
         D_pur_house=ifelse(purpose=='house',1,0),
         D_pur_renewable=ifelse(purpose=='renewable_energy',1,0),
         D_pur_edu=ifelse(purpose=='educational',1,0),
         annual_inc=annual_inc/10000)

cs <- cs[complete.cases(cs),]

set.seed(12345)
rows <- sample(1:42506,8501,replace=F)
cs_train <- cs[-rows,]
cs_val <- cs[rows,]

```

# Predictions

## Logit Model

In R logistic regression is accomplished by specifying a generalized linear model with a binomial link function.  Out-of-sample predictions are obtained by using the *predict.glm* wrapper and supplying a new data frame with covariates upon which the predictions will be made

```R
#logit with dummy variables relative to car purchase
logit <- glm(paid~int_rate+dti+annual_inc+delinq_2yrs+
               D_pur_credit + D_pur_smallbiz + D_pur_other + D_pur_wedding + 
               D_pur_debt + D_pur_homeimp + D_pur_majorpurch + D_pur_medical + 
               D_pur_moving + D_pur_vacation + D_pur_house + D_pur_renewable +
               D_pur_edu,family="binomial",data=cs_train)

#--------------------------------------------------
# predict from the validation set and calculate
# % correctly predicted
logit.pred <- predict.glm(logit,cs_val,type="response")

cs_val <- cs_val %>% mutate(yhat=logit.pred,
                            class=ifelse(yhat>0.5,1,0),
                            pred.correct = as.numeric(class==paid))

(sum(cs_val$pred.correct))/nrow(cs_val)

0.842

```

We can see that about 85% of the observations in the out-of-sample data were correctly predicted.

## LDA

A 'black box' for linear discriminant analysis is available in the *MASS* package.  The functions are pretty straightforward:

* lda(formula, data) cranks out the coefficients of the discriminating function
* predict(object,data) provides the predicted groupings for the new data

```R

#specify an LDA object
cs_lda <- lda(paid~int_rate+dti+annual_inc+delinq_2yrs+
             D_pur_credit + D_pur_smallbiz + D_pur_other + D_pur_wedding + 
             D_pur_debt + D_pur_homeimp + D_pur_majorpurch + D_pur_medical + 
             D_pur_moving + D_pur_vacation + D_pur_house + D_pur_renewable +
             D_pur_edu, data=cs_train)

lda.pred <- predict(cs_lda,cs_val)$class
cs_val <- cs_val %>% mutate(lda.pred=lda.pred, lda.correct=as.numeric(lda.pred==paid))

#percent correctly predicted with LDA
(sum(cs_val$lda.correct))/nrow(cs_val)
0.8409

```

Here we see that LDA also correctly predicts the outcome of about 85% of our out-of-sample data.

# Marginal Effects

Here's something you get with logistic regression that you don't get with Linear Discriminant Analysis: an estimate of the marginal impact of a covariate.

As far as I know (and I could totally be wrong here) I don't think a traditional Linear Discriminant Analysis in the way Fisher conceived of it is capable of generating an estimate of the impact of an incremental change in a predictor variable on the probability of observing an outcome.

For more traditional data mining problems, like correctly classifying an Iris based on some physical characteristics, marginal effects probably don't matter much.  At least to me, it doesn't make a lot of intuitive sense to say,

"holding all else equal what is the change in probability of classifying an Iris as Setosa versus Virginica or Versacolour from a 1 unit change in petal length."

But for something like credit risk it seems kind of important to me to try and understand what factors drive default risk...so asking,

"what is the change in the expected probability of default resulting from a hypothetical one unit change in age or annual income," seems to me to be a pretty obvious thing to care about.

```R
margins(logit, at = list(int_rate = mean(cs$int_rate,na.rm=T), 
                     dti = mean(cs$dti,na.rm=T),
                     delinq_2yrs=mean(cs$delinq_2yrs,na.rm=T),
                     annual_inc=mean(cs$annual_inc,na.rm=T),
                     D_pur_credit=0,
                     D_pur_smallbiz=0,
                     D_pur_other=0,
                     D_pur_wedding=0,
                     D_pur_debt=1,
                     D_pur_homeimp=0,
                     D_pur_majorpurch=0,
                     D_pur_medical=0,
                     D_pur_moving=0,
                     D_pur_vacation=0,
                     D_pur_house=0,
                     D_pur_renewable=0,
                     D_pur_edu=0))


```
