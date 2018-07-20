so I had some fun messing around with a new [R package for doing AB Testing](https://cran.r-project.org/web/packages/bayesAB/index.html) in a Bayesian framework. Here's my report:

# AB Testing
AB testing appears to me to be yet another way that private sector folks have tried to rebrand basic statistics as something they invented.  Disclaimer: I'm no expert and the jargon of 'AB Testing' might have a long history in the scientific literature.  

The nub of the problem is this: 

* we have a current state of affairs (a red pill, 14 point font on the 'click here' button for a website, free school lunch) - the 'A' case
* we have an alternative treatment (a blue pill, 12 point font on the 'click here' button, etc)
* we want to design a controlled experiment to determine whether B is better than A

To fix ideas, the canonical problem we will consider here is this:

* we have a webpage with a 'become a member' button, we think we have some idea what the current success rate (the proportion of visitors who become members) is.
* we are considering a new page design and want to know if the new design will increase the success rate

# The Frequentist Way

I'm gonna be pretty hand-wavey here because there are literally a thousand places you can go for primers on experimental design in the frequentist context.  The basics are this:

Suppose we think the current success rate is 30%.  We have a claim (from our product team) that the new page design will increase the success rate to 40%.  In the frequentist universe we would:

* decide on a confidence level for our test (say $\alpha$ = 95%)
* decide on a power level (the probability of correctly detecting an effect size of 30%) for our test (say 80%)
* use the standard power calculators to determine how many study subjects in each group we need in order to be 95% confident the observed success rate for group B is 30% higher than the observed success rate for group A.  

# Some Bayesian Basics

The backbone of the frequentist methodology is rejecting the null hypothesis of 0 effect.  I don't want to get into a big thing about frequentists vs. Bayesian but, for the purposes of this post I will point out that many people find the interpretability of the frequentist approach unappealing.

The Bayesian approach proceeds by treating the underlying parameter value (the success rate) as a random variable with a distribution that can be inferred from the data...This contrasts with the frequentist approach of assuming the underlying population parameter value is fixed and all the uncertainty comes from sampling error.

In our example of Bayesian AB testing the quantity of most intense interest is the posterior distribuiton of the success rate.  

Bayes Rule says:

$P(\theta|X)=\frac{P(X|\theta)P(\theta)}{P(X)}$

1.  $P(\theta|X)$ is the prior probability of observing the parameter $\theta$ given the data, $X$.
2.  $P(X|\theta)$ is the likelihood of observing the data sample $X$ given the parameter $\theta$.
3.  $P(X)$ is the marginal probability of observing the data sample $X$.

This can be leveraged for AB testing in the following way.  Suppose we belive the current success rate is 0.3.  We want to test whether some webpage improvements will get us a success rate of 0.4.  

In the Bayesian sense what we would like to do is show a bunch of people the original page and estimate the posterior distribution of the success rate.  Then show a bunch of people the new webpage and estimate the posterior distribution of that success rate.  Comparing the posterior distributions of two success rates will tell us whether we belive the success rate for the new page is better than the success rate for the old page.

The mechanics of Bayesian Stats are a little intimidating but the basic idea for AB testing can be distilled like so:

Given the current success rate of 0.3 and a data sample of a few hundred people who were shown the first page, the prior distribution of success rate is:

$P(\theta=0.3|X)=\frac{P(\theta = 0.3)P(X|\theta=0.3)}{P(X)}$

Let's start with the likelihood:

Each individual either signs up or doesn't so the likelihood of the data is a Bernoulli distribution with success rate $\theta$:

$P(X|\theta) = \Pi_{i}^{N}\theta^{x_{i}}(1-\theta)^{1-x_{i}}$

The prior is a little more complicated but let's gloss over the complexity for now and just accept that we want a conjugate prior for the binomial distribution and sources tell us that the conjugate distribution for the binomial is the Beta distribution.  So we will set our prior on $\theta$ to be,

$\theta ~ Beta(a,b)=\frac{\theta^{a-1}(1-\theta)^{b-1}}{B(a,b)}$

where $B(a,b)$ is the Beta function.




# Some Bayesian Illustrations

For our numerical example we stated that we belive $\theta$ to be around 0.3.  Let's look at a beta distribution centered around 0.3:

```{r}
hist(rbeta(100,5,5))
hist(rbeta(100,50,50))
hist(rbeta(1000,0.3*1000,0.7*1000))
```
The variance of the beta distribution is controlled by the size of the shape parameters a and b.

# Quick Look at the bayesAB package
