
I know I said last week's post would be my final words on Twitter Mining/Sentiment Analysis/etc. for a while.  I guess I lied.  I didn't feel great about the black box-y application of text classification...so I decided to add a little 'under the hood' post on Naive Bayes for text classification/sentiment analysis.

I've already covered the quick and dirty theoretical/conceptual elements of the Naive Bayes classifier [here](https://aaronmams.github.io/Sentiment-Analysis-with-Python-Part-2/).  In this post I'm going to try and walk through a toy example with some numerical meat to the theoretical treatment.

Also, be warned, I'm reverting back to my R roots here because I'm not yet fluent enough with Python to do a lot of the quick off-the-cuff 
coding I want to include here.

## First Pass

Here is some preliminary nomenclature:

* I have a statement that I want to classify as positive or negative.  I call this a document and I distinguish it from other documents (such as those that will populate the training data) with the subscript j: $$d_{j}$$.
* I have a bunch of statements that have been classified as 'positive' and 'negative' and I call the the training set, $$D$$.
* The universe of possible classification is denoted as $$C$$. And $$C$$ can take the values $$p$$ for positive and $$n$$ for negative.

The statement $$d_{j}$$ will be classified as either $$p$$ or $$n$$ depending on which class has a higher posterior probability:

The posterior probability that $$d_{j}$$ is positive is,

$$P(C=p|d_{j})=\frac{P(d_{j}|C=p)P(C=p)}{P(d_{j})}$$

The posterior probability that $$d_{j}$$ is negative is,

$$P(C=n|d_{j})=\frac{P(d_{j}|C=n)P(C=p)}{P(d_{j})}$$

As discussed previously, the denomenator turns out to be irrelevant in the calculation since it's the same in both quantities.  Therefore the key quantities necessary to evaluate the posterior probabilities are:

$$P(d_{j}|C)$$ 

the likelihood of the document, and

$$P(C)$$ 

the prior probability of observing a document class.

In the simplest case of conditional independence that we have been operating under (meaning that the occurance of words in a document is independent of other words in the document), the probability of observing document $$j$$ in class $$i$$ can be expressed as a combination of the probabilities of observing the words in document $$j$$, ($$w$$) conditional on the document belonging to class $$i$$.  More specifically, the likelihood that document $$j$$ is positive is,

$$P(d_{j}|C=p)=\Pi_{t}[b_{jt}P(w_{t}|C=p)+(1-b_{it})(1-P(w_{t}|C=p))]$$

In the equation above, $$b_{jt}$$ is a [0,1] indicator for whether word, $$w_{t}$$ appears in document $$j$$ or not.  

### Word Likelihoods

The individual components of the overall document likelihood 

$$P(w_{t}|C=p)$$ 

in the simple model, are approximated by relative frequencies:

$$P(w_{t}|C=p)=\frac{n_{p}(w_{t})}{N_{p}}$$

where $$n_{p}(w_{t})$$ is the number of documents in class $$p$$ where $$w_{t}$$ appears.  And $$N_{p}$$ is the total number of positively classified document.

### The Priors 

The prior probability of observing a document classified as positive or negative can be parameterized using the relative frequencies of document of different classes:

$$P(C=p)=\frac{N_{p}}{N}$$

where $$N_{p}$$ is the total number of documents classified as positive and $$N$$ is the total number of documents.  

## An Example

In the example below I do the following:

* start with a training set of documents that have been classified as positive or negative. 
* then create a vocabulary (sometimes called the 'feature set') which is just a listing of all the words in the documents in the training set.  
* then, for every word in the vocabulary, tabulate the number 'positive' documents containing that word divided by the total number of positive documents and the number of 'negative' documents containing the word divided by the total number of negative documents.   


```R
train.df <- data.frame(D=c('this book is awesome',
                           'harry potter books suck',
                           'these pretzles are making me thirsty',
                           'they choppin my fingers off Ira',
                           'supreme beings of leisure rock',
                           'cheeto jesus is a tyrant'
                           ),
                       Sent=c('positive',
                              'negative',
                              'negative',
                              'negative',
                              'positive',
                              'negative'))
#create vocabulary
words <- data.frame(w=unlist(strsplit(as.character(train.df$D)," ")))
words.i.dont.want <- c('a','this','me','are','of','is','my','these','they')
words <- tbl_df(words) %>% filter(!w %in% words.i.dont.want)

#some intermediate vocabs
positive.docs <- strsplit(as.character(train.df$D[which(train.df$Sent=='positive')])," ")
negative.docs <- strsplit(as.character(train.df$D[which(train.df$Sent=='negative')])," ")

#find the number of positive docs containing each word
npos <- function(w){sum(as.numeric(unlist(lapply(positive.docs,function(x){w %in% x}))))}
nneg <- function(w){sum(as.numeric(unlist(lapply(negative.docs,function(x){w %in% x}))))}

n_pos<- unlist(lapply(unique(words$w),npos))
n_neg <- unlist(lapply(unique(words$w),nneg))

words$n_pos <- n_pos
words$n_neg <- n_neg

#for each word you can get P_hat(w|C=pos) and P_hat(w|C=neg)
words <- words %>% mutate(p_hat_pos=(n_pos)/nrow(train.df[train.df$Sent=='positive',]),
                          p_hat_neg=(n_neg)/nrow(train.df[train.df$Sent=='negative',]))

#prior on positive vs. negative is just the relative frequency of each
prior.pos <- nrow(train.df[train.df$Sent=='positive',])/nrow(train.df)
prior.neg <- nrow(train.df[train.df$Sent=='negative',])/nrow(train.df)

> print.data.frame(words)
          w n_pos n_neg p_hat_pos p_hat_neg
1      book     1     0       0.5      0.00
2   awesome     1     0       0.5      0.00
3     harry     0     1       0.0      0.25
4    potter     0     1       0.0      0.25
5     books     0     1       0.0      0.25
6      suck     0     1       0.0      0.25
7  pretzles     0     1       0.0      0.25
8    making     0     1       0.0      0.25
9   thirsty     0     1       0.0      0.25
10  choppin     0     1       0.0      0.25
11  fingers     0     1       0.0      0.25
12      off     0     1       0.0      0.25
13      Ira     0     1       0.0      0.25
14  supreme     1     0       0.5      0.00
15   beings     1     0       0.5      0.00
16  leisure     1     0       0.5      0.00
17     rock     1     0       0.5      0.00
18   cheeto     0     1       0.0      0.25
19    jesus     0     1       0.0      0.25
20   tyrant     0     1       0.0      0.25

#classify an unlabeled document:
#note: we won't be able to compute the posterior probability
# of this document because the word 'cheeto' is not observed in
# any positive training documents...and the word 'awesome' is 
# not observed in any negative training document:

d_j <- 'just had my first cheeto ever it was awesome'

```

So this is actually an example of something that doesn't work.  Here's the reason:

We want to calculate the posterior probability that document $$j$$ is positive and compare that to the posterior probability that document $$j$$ is negative.  Let's start by defining document $$j$$ as a binary vector with entries equal to the number of entries in the vocabulary (in the code above this is the data frame 'words').  So the document that I want to classify ('just had my first cheeto ever it was awesome') can be represented by the vector:

$$d_{j} = \begin{pmatrix}0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0\end{pmatrix}$$ 

and

$$(1-d_{j}) = \begin{pmatrix}1,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,1,1\end{pmatrix}$$

and the class conditional probabilities based on relative frequencies of appearance in positive documents is:

$$P(w_{t}|C=positive) = \begin{pmatrix} 0.5,0.5,0,0,0,0,0,0,0,0,0,0,0,0.5,0.5,0.5,0.5,0,0,0\end{pmatrix}$$ 

and

$$P(w_{t}|C=negative) = \begin{pmatrix} 0,0,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0,0,0,0,0.25,0.25,0.25\end{pmatrix}$$

which yields the following likelihood calculation:

$$P(d_{j}|C=positive)=(0(0.5)+1(0.5))(1(0.5)+0(0.5))(0(0)+1(1))...(1(0)+0(1))...=0$$

$$P(d_{j}|C=negative)=(0(0)+1(1))(1(0)+0(1))(0(0.25)+1(0.75)).....=0$$

The thing about this problem is that, because the feature 'cheeto' is not observed in any documents classified 'positive' in the training set, the calculation for the likelihood of a new document containing the feature 'cheeto' being 'positive' falls apart.  The same is true of the likelihood of a new document containing the feature 'awesome' being 'negative'...because there are no instances in the training set where 'awesome' appears in a 'negative' document.

### Two Fixes 

One way to deal with the absent feature problem is [to add a smoothing term](http://sebastianraschka.com/Articles/2014_naive_bayes_1.html) to the probability of observing word $$t$$ in a document of class 'positive':

$$P(w_{t}|C=positive)=\frac{n_{p}(w_{t} + \alpha)}{N_{p}+\alpha}$$

The second fix is waaaaay more ghetto but it serves my current purpose (quick under-the-hood illustration of Naive Bayes) a little better: let's just add one positive document with the word 'cheeto' in it and one one negative document with the word 'awesome' in it to the training data:

```R
train.df <- data.frame(D=c('this book is awesome',
                           'the book is awesome, jk',
                           'harry potter books suck',
                           'these pretzles are making me thirsty',
                           'they choppin my fingers off Ira',
                           'supreme beings of leisure rock',
                           'cheeto jesus is a tyrant',
                           'jesus awesome cheeto'
                           ),
                       Sent=c('positive',
                              'negative',
                              'negative',
                              'negative',
                              'negative',
                              'positive',
                              'negative',
                              'positive'))
#create vocabulary
words <- data.frame(w=unlist(strsplit(as.character(train.df$D)," ")))
words.i.dont.want <- c('a','this','me','are','of','is','my','these','they','jk')
words <- tbl_df(words) %>% filter(!w %in% words.i.dont.want)

```
Assuming our classifier is sophisticated enough to recognize the 'jk' and classify the second document as negative then the following minimal changes result:

$$d_{j} = \begin{pmatrix}0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0\end{pmatrix}$$ 

and

$$(1-d_{j}) = \begin{pmatrix}1,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,1,1\end{pmatrix}$$

and the class conditional probabilities based on relative frequencies of appearance in positive documents is:

$$P(w_{t}|C=positive) = \begin{pmatrix} 0.33,0.66,0,0,0,0,0,0,0,0,0,0,0,0.33,0.33,0.33,0.33,0,0.33,0\end{pmatrix}$$ and

$$P(w_{t}|C=negative) = \begin{pmatrix} 0.2,0.2,0.2,0.2,0.2,0.2,0.2,0.2,0.2,0.2,0.2,0.2,0.2,0,0,0,0,0.2,0.2,0.2\end{pmatrix}$$

which yields the following likelihood calculation:

$$P(d_{j}|C=positive)=(0(0.33)+1(0.666))(1(0.66)+0(0.33))(0(0)+1(1))...(1(0.33)+0(0.66))...\sim 0.019$$

$$P(d_{j}|C=negative)=(0(0.2)+1(0.8))(1(0.2)+0(0.8))...((1(0.2)+0(0.8))())).....\sim 0.002$$

To get the posterior probability that our classification will be based on, we need to combine these likelihoods and the priors:

$$P(C=positive|d_{j})=P(d_{j}|C=positive)P(C=positive)=$$

$$0.019(\frac{3}{8})=0.007$$.

and

$$P(C=negative|d_{j})= P(d_{j}|C=negative)P(C=negative)=$$

$$0.002(\frac{5}{8})=0.001$$.

The complete script used to recover these quantities is

```R
library(dplyr)

#define a training set:
train.df <- data.frame(D=c('this book is awesome',
                           'this book is awesome jk',
                           'harry potter books suck',
                           'these pretzles are making me thirsty',
                           'they choppin my fingers off Ira',
                           'supreme beings of leisure rock',
                           'cheeto jesus is a tyrant',
                           'jesus awesome cheeto'
                           ),
                       Sent=c('positive',
                              'negative',
                              'negative',
                              'negative',
                              'negative',
                              'positive',
                              'negative',
                              'positive'))
#create vocabulary
words <- data.frame(w=unique(unlist(strsplit(as.character(train.df$D)," "))))
words.i.dont.want <- c('a','this','me','are','of','is','my','these','they','jk')
words <- tbl_df(words) %>% filter(!w %in% words.i.dont.want)

#some intermediate vocabs
positive.docs <- strsplit(as.character(train.df$D[which(train.df$Sent=='positive')])," ")
negative.docs <- strsplit(as.character(train.df$D[which(train.df$Sent=='negative')])," ")

#find the number of positive docs containing each word
npos <- function(w){sum(as.numeric(unlist(lapply(positive.docs,function(x){w %in% x}))))}
nneg <- function(w){sum(as.numeric(unlist(lapply(negative.docs,function(x){w %in% x}))))}

n_pos<- unlist(lapply(unique(words$w),npos))
n_neg <- unlist(lapply(unique(words$w),nneg))

words$n_pos <- n_pos
words$n_neg <- n_neg

#for each word you can get P_hat(w|C=pos) and P_hat(w|C=neg)
words <- words %>% mutate(p_hat_pos=(n_pos)/nrow(train.df[train.df$Sent=='positive',]),
                          p_hat_neg=(n_neg)/nrow(train.df[train.df$Sent=='negative',]))

#prior on positive vs. negative is just the relative frequency of each
prior.pos <- nrow(train.df[train.df$Sent=='positive',])/nrow(train.df)
prior.neg <- nrow(train.df[train.df$Sent=='negative',])/nrow(train.df)

#classify an unlabeled document:
#note: we won't be able to compute the posterior probability
# of this document because the word 'cheeto' is not observed in
# any positive training documents...and the word 'awesome' is 
# not observed in any negative training document:

d_j <- 'just had my first cheeto ever it was awesome'

#create the indicator vector for words in the vocabulary
dj <- unlist(strsplit(as.character(d_j)," "))

words$b <- as.numeric(words$w %in% dj)
words <- words %>% mutate(l_pos=(b*p_hat_pos)+((1-b)*(1-p_hat_pos)),
                          l_neg=(b*p_hat_neg)+((1-b)*(1-p_hat_neg)))

#finally, get the posterior probs by combining the prior and likelihoods
post_pos <- (3/8)*prod(words$l_pos)
post_neg <- (5/8)*prod(words$l_neg)

> post_pos
[1] 0.007315958

> post_neg
[1] 0.001099512

```


## Summary

For each word in the document we are trying to classify there is one positive training document and one negative training document containing the word...Why aren't the probabilities 50/50? 

The best way to conceptualize this result is probably by noting that 

* there is one positive document containing the word 'awesome' and one negative document
* there one positive and one negative document containing the word cheeto
* there are 3/8 total documents classified 'positive'
* there are 5/8 total documents classified 'negative'

Because there are more 'negative' documents than 'positive' documents the naive prior probability of a document being negative is higher: 5/8 vs. 3/8.  

However there is a greater share of positive documents containing the word 'awesome' than negative documents containing the word 'awesome' (1 out of 3 verses 1 out of 5) and there is a greater share of positive documents containing the word 'cheeto' than negative documents containing the word 'cheeto' (again 1 out 3 versus 1 out of 5)...so the likelihood of D=positive conditional on the data is slightly higher than the likelihood of D=negative conditional on the data.  


## Appendix

In [a previous post here](https://aaronmams.github.io/Sentiment-Analysis-with-Python-Part-2/) I had to do some serious trimming to get my classifier trained.  In that example I started with approximately 1 million tweets (Document) and didn't strip out very many words from the tweets so the vocabulary or dictionary or whatever you want to call it (the list of words used to predict 'positive' or 'negative' for a new observation was really big...like several million.

From this post one can see that procedurally - at least for the simple Naive Bayes Bag of Words Model - the computationally demanding part of the algorithm is getting the relative frequencies of each word in each class:

$$P(w_{t}|C=p)=\frac{n_{p}(w_{t})}{N_{p}}$$

I wanted to get some idea of whether the wait times I was experiencing with Python's NLTK were due to user error, black box algorithm inefficiency,...or if I ought to just expect the training phase of a simple sentiment analysis to take a while.  So I set up a little simulation:

* I used a corpus from R's 'tm' package to get a 'bag of words'
* I used a random sample of 3 words from this bag to define a set of documents
* I assigned half of the documents the sentiment 'positive' and half of them the sentiment 'negative.'
* For each word in the bag I calculated the number of positive documents containing the word and number of negative documents containing the word.

### Two simulations:

First I only used the words appearing in the data list 'crude' from the 'tm' package in R.  I removed some stop words and nonsense words, which resulted in a final set of 1,085 words.  I created 100,000 3-word combinations of these words and assigned half to be 'positive' and half to be 'negative.'    

```R
#load libraries
library(tm)

#load data
data(crude)

#clean up word list
words.crude <- list()
for(i in 1:20){
  x <- crude[[i]]$content
  x <- gsub('\n'," ",x)
  x<- gsub("[[:punct:]]", " ", x)
  x <- strsplit(x," ")
  words.crude[[i]] <- x
}
words.crude <- unlist(words.crude)
words.crude <- tolower(words.crude)

words.crude <- words.crude[which(!words.crude %in% c("","is","a","the","an","this","he","she","it","in","to","its","of"))] 
words.crude <- unique(words.crude)

#create a list of 100,000 simulated documents...each document containing 3 randomly sampled
# words from the word list
vocab <- list()
for(i in 1:100000){
  xtmp <- paste(sample(words.crude,1),sample(words.crude,1),sample(words.crude,1),sep=" ")
  vocab[[i]] <- xtmp
}

vocab <- data.frame(as.matrix(unlist(vocab),nrow=100000))

#assign half of the simulated documents to be 'positive' and half 'negative'
vocab$Sent <- c(rep('positive',50000),rep('negative',50000))
names(vocab) <- c('D','Sent')

#keep only the unique words
v <- data.frame(w=unique(unlist(strsplit(as.character(vocab$D)," ")))) 

#some intermediate vocabs
positive.docs <- strsplit(as.character(vocab$D[which(vocab$Sent=='positive')])," ")
negative.docs <- strsplit(as.character(vocab$D[which(vocab$Sent=='negative')])," ")

#create a matrix with # of columns = # of words in the vocabulary and
# number of rows equal to number of documents...the (i,j) element is 1 if
# word j appears in document i and 0 otherwise.  Summing over the columns
# of each matrix give us the number of 'positive' document containing each word
# and number of 'negative' documents containing each word.
t <- Sys.time()
z <- lapply(positive.docs,function(x)which(v$w %in% x))
#z <- matrix(0,nr=50000,nc=length(v$w))
pos.mat <- lapply(1:50000,function(x){
  r=rep(0,length(v$w))
  idx <- c(z[[x]])
  r[idx]<-1
  return(r)
  }
)
pos.mat <- matrix(c(unlist(pos.mat)),nr=50000,nc=length(v$w),byrow=T)
v$n_pos <- apply(pos.mat,2,FUN=sum)

z <- lapply(negative.docs,function(x)which(v$w %in% x))
#z <- matrix(0,nr=50000,nc=length(v$w))
neg.mat <- lapply(1:50000,function(x){
  r=rep(0,length(v$w))
  idx <- c(z[[x]])
  r[idx]<-1
  return(r)
}
)
neg.mat <- matrix(c(unlist(neg.mat)),nr=50000,nc=length(v$w),byrow=T)
v$n_neg <- apply(neg.mat,2,FUN=sum)
Sys.time() - t

Time difference of 16.3339 secs
```



The second simulation I doubled the number of unique words in the vocabulary to 2,000 and I doubled the number of documents in the training set to 200,000 documents each containing three randomly selected words from the vocabulary.  I again made half of these training documents positive and half of them negative.

My goal here was to see how the training time changed when I increased the size of the problem considerably.


```R
wordlist = list()
for(i in 1:500){
 c <- sample(letters,4)
 wordlist[[i]] <- paste(c[1],c[2],c[3],c[4],sep="")
} 
 
for(i in 501:1000){
  c <- sample(letters,5)
  wordlist[[i]] <- paste(c[1],c[2],c[3],c[4],c[5],sep="")
} 

for(i in 1001:2000){
  c <- sample(letters,6)
  wordlist[[i]] <- paste(c[1],c[2],c[3],c[4],c[5],c[6],sep="")
} 

wordlist <- c(unique(unlist(wordlist)))

vocab <- list()
for(i in 1:200000){
  xtmp <- paste(sample(wordlist,1),sample(wordlist,1),sample(wordlist,1),sep=" ")
  vocab[[i]] <- xtmp
}

vocab <- data.frame(as.matrix(unlist(vocab),nrow=200000))
vocab$Sent <- c(rep('positive',100000),rep('negative',100000))
names(vocab) <- c('D','Sent')

v <- data.frame(w=unique(unlist(strsplit(as.character(vocab$D)," ")))) 

#some intermediate vocabs
positive.docs <- strsplit(as.character(vocab$D[which(vocab$Sent=='positive')])," ")
negative.docs <- strsplit(as.character(vocab$D[which(vocab$Sent=='negative')])," ")


#try a matrix with column sums version
t <- Sys.time()
z <- lapply(positive.docs,function(x)which(v$w %in% x))
#z <- matrix(0,nr=50000,nc=length(v$w))
pos.mat <- lapply(1:100000,function(x){
  r=rep(0,length(v$w))
  idx <- c(z[[x]])
  r[idx]<-1
  return(r)
}
)
pos.mat <- matrix(c(unlist(pos.mat)),nr=100000,nc=length(v$w),byrow=T)
v$n_pos <- apply(pos.mat,2,FUN=sum)

z <- lapply(negative.docs,function(x)which(v$w %in% x))
#z <- matrix(0,nr=50000,nc=length(v$w))
neg.mat <- lapply(1:100000,function(x){
  r=rep(0,length(v$w))
  idx <- c(z[[x]])
  r[idx]<-1
  return(r)
}
)
neg.mat <- matrix(c(unlist(neg.mat)),nr=100000,nc=length(v$w),byrow=T)
v$n_neg <- apply(neg.mat,2,FUN=sum)
Sys.time() - t
Time difference of 53.67482 secs
```

So these preliminary results here suggest I can train a Naive Bayes , Bag of Words, positive/negative classifier with 100,000 training documents and 1,000 unique words in around 20 seconds.  If I double the number of unique words and double the number of training samples I increase training time to around a minute.  

This makes me a little suspicious of the 15 minute training times I was getting using Python's NLTK with 100,000 tweets and several thousand unique words in the training data.  

Before closing this post out, it's probably worth noting that my solution here of creating a matrix of 0's and 1's that can be summed column-wise to get class condtional word frequencies isn't idea for scaling.  I don't know this for fact...but I'm pretty sure that R treats matricies as vectors so the theoretical limit would be something like:

$$ (nrow)(ncol) < 2^{31-1}$$

in practice my instance of R Studio run on a pretty powerful workstation craps out somewhere around 1,000,000 rows and 1,500 columns.

But even the least efficient case of looping over words and counting the number of documents containing that word should be relatively easy to parallelize....I'll have to test that theory another time.  
