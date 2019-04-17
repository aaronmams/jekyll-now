
I have a GitHub Repo set up to accompany this post.  It's here:

[https://github.com/aaronmams/rad-teaching-stuff/tree/master/caret-example](https://github.com/aaronmams/rad-teaching-stuff/tree/master/caret-example)

I have a couple older posts on pipelines and, more specifically, Machine Learning pipelines. They are:  

[Here is a pipeline post using R](https://aaronmams.github.io/Machine-learning-pipelines-part-1/).  It's mostly a pipeline that I rolled myself.  There are plenty of library dependencies but the actual chaining of data retrieval, data preprocessing, feature selection, model selection is done mostly by hand.

[Here is the first of python-based posts](https://aaronmams.github.io/Machine-Learning-Pipelines-3-Data-streams/) I wrote on data mining and machine learning pipelines.

In this post I want to revisit the topic of Machine Learning Pipelines in R.  Specifically, I'm going to present some examples of training machine learning models using the [caret package](https://topepo.github.io/caret/index.html).

*caret()* provides a cool wrapper method called *train()* that allows analysts to access models from a variety of different machine learning libraries.  

# Key Observations

1. The *nnet()* method actually performs [one-hot-encoding](https://www.kaggle.com/dansbecker/using-categorical-data-with-one-hot-encoding) behind the scenes.  This isn't some revolutionary discovery...but it is something that hung me up for a few minutes, isn't really documented well, and, since I believe modelers should now what their models are doing mechanically as well as conceptually, it something I think people should know.

2. Some data pre-processing can be done inside the call to the model training method *train()*, but I prefer doing pre-processing operations before the call to *train()* since this seems more transparent to me.

3. 

# Problem Set Up

I'm going to work with some satellite tracking data I have on animal feeding. [The data can be downloaded from GitHub](https://github.com/aaronmams/rad-teaching-stuff/tree/master/caret-example). These data provide georeferenced coordinates for each animal's position every hour.  Here's what they look like:

```r
> str(dat)
Classes ‘tbl_df’, ‘tbl’ and 'data.frame':	179718 obs. of  10 variables:
 $ X         : int  1 2 3 4 5 6 7 8 9 10 ...
 $ foraging  : int  0 0 0 0 0 0 0 0 0 0 ...
 $ speed     : num  0 0 0 0 0 0 0 0 0 0 ...
 $ depth     : num  112 112 112 112 112 ...
 $ angle     : num  0 0 0 0 0 0 0 0 0 0 ...
 $ utc_date  : POSIXct, format: "2014-01-02 00:52:00" "2014-01-02 01:52:00" "2014-01-02 02:52:00" ...
 $ local_time: POSIXct, format: "2014-01-01 16:52:00" "2014-01-01 17:52:00" "2014-01-01 18:52:00" ...
 $ animalID  : int  1 1 1 1 1 1 1 1 1 1 ...
 $ hour      : Factor w/ 4 levels "1","2","3","4": 3 3 4 4 4 4 4 4 1 1 ...
 $ length    : Factor w/ 5 levels "1","2","3","4",..: 3 3 3 3 3 3 3 3 3 3 ...
```  

* foraging in a [0,1] indicator for whether the animal was foraging or not
* speed is the instaneous speed of the animal at each observation point
* depth is the ocean bottom depth associate with each location
* angle is the bearing in radians of each position
* utc_date is the date-time in universal time
* local_time is the utc_date converted to Pacific time
* hour is the hour of the day which has been coded as a factor with 4 levels
* length is the length of the animal which has been coded as a factor variable with 5 levels

I prefer not to get too far down a rabbit hole of ecological theory here since this is really just a toy problem to illustrate how to train largely atheoretic machine learners...but, in the interest of not being completely robotic, we could hypothsize that:

Animals might move at very low speeds when resting or sleeping (two possible non-foraging behaviors) and move at much higher speeds when migrating (another possible non-foraging behavior).  The animals might move at sort of in-between speeds when foraging.  In our data, which may or may not be representative of any actual animal, this patter is apparent:

![speeds](/images/foraging-speed.png)

We might also image that depth (we'll pretend these are marine predators) is a proxy for habitat and mayb the animals prefer to forage in certain types of habitat.  Length will help control for heterogeneity (maybe different sized animals forage at different speeds or prefer different habitat), hour-of-day could capture potential preferences for foraging at night versus during the day, etc. 


```r
> summary(dat)
       X             foraging          speed              depth               angle       
 Min.   :     1   Min.   :0.0000   Min.   :    0.00   Min.   :   0.0002   Min.   :0.0000  
 1st Qu.: 44930   1st Qu.:0.0000   1st Qu.:    0.02   1st Qu.:  59.1099   1st Qu.:0.1365  
 Median : 89860   Median :0.0000   Median :    3.72   Median : 181.8442   Median :2.6130  
 Mean   : 89860   Mean   :0.2794   Mean   :    6.35   Mean   : 282.4487   Mean   :2.5430  
 3rd Qu.:134789   3rd Qu.:1.0000   3rd Qu.:    7.65   3rd Qu.: 464.5657   3rd Qu.:4.4097  
 Max.   :179718   Max.   :1.0000   Max.   :45209.96   Max.   :3077.3398   Max.   :6.2830  
    utc_date                     local_time                     animalID   hour      length   
 Min.   :2014-01-01 08:13:00   Min.   :2014-01-01 00:13:00   Min.   :  1   1:45853   1:24630  
 1st Qu.:2014-04-30 22:40:00   1st Qu.:2014-04-30 15:40:00   1st Qu.: 39   2:45808   2:49639  
 Median :2014-07-21 22:29:00   Median :2014-07-21 15:29:00   Median : 66   3:43467   3:86372  
 Mean   :2014-07-08 04:47:42   Mean   :2014-07-07 21:47:42   Mean   : 63   4:44590   4:14155  
 3rd Qu.:2014-09-12 17:04:24   3rd Qu.:2014-09-12 10:04:24   3rd Qu.: 91             5: 4922  
 Max.   :2014-12-31 23:50:00   Max.   :2014-12-31 15:50:00   Max.   :113 
```

My goal here it to be able to predict whether an animal is foraging or not given it's location and some other physical/environmental characteristics associated with its location. I'm going to fit 2 classes of models to my training data: 

1. An Artificial Neural Network (ANN)

2. A Gradient Boosting Machine (GBM)

I'm going to use the functionality in [caret](https://topepo.github.io/caret/index.html) to find the best ANN and the best GBM to predict the behavioral state of unlabeled observations.


# The Models:

## Really simple:


## A little harder


# CARET

Using the caret library to train machine learning models, there are a few choices we need to make:

First, caret's *train()* function includes a parameter for what model you want to train.  [Check out Chapter 6 here](https://topepo.github.io/caret/available-models.html) for the full list of models and libraries to choose from.  These include, but are not limited to: Adaptive Boosting Classification Trees from the [fastAdaboost library](https://cran.r-project.org/web/packages/fastAdaboost/index.html), Bayesian Adaptive Regression Trees from the [bartMachine library](https://cran.r-project.org/web/packages/bartMachine/index.html), Neural Networks from the [nnet library](https://cran.r-project.org/web/packages/nnet/nnet.pdf), Neural Networks from the [neuralnet library](https://cran.r-project.org/web/packages/neuralnet/neuralnet.pdf), Generalized Partial Least Squares from [gpls library](https://bioconductor.org/packages/release/bioc/html/gpls.html)...and literally about 600 more.

For my first class of model I've choosen ANNs but now I need to choose which library and method value I'm going to use.  In this case, I'm choosing the [nnet() method](https://cran.r-project.org/web/packages/nnet/nnet.pdf) from the *nnet* library.  The second model class that I'm experimenting with is Gradient Boosting Machines.  Again, there are a couple of libraries in R that will do this flavor or learner...I'm going with Stochastic Gradient Boosting via the *gbm()* method in the [gbm library](https://cran.r-project.org/web/packages/gbm/gbm.pdf).  

## First Pass

```r
# A proper CV experiment with ANNs

#--------------------------------------------------------------------
#create the training and testing data
index <- createDataPartition(df$y,p=0.8,list=F,times=1)
df.train <- df[index,] %>% mutate(y=as.factor(y))
levels(df.train$y) <- c('notforaging','foraging') 
df.test <- df[-index,] %>% mutate(y=as.factor(y))
levels(df.test$y) <- c('notforaging','foraging')  
#--------------------------------------------------------------------

#--------------------------------------------------------------------
# set up control parameters for the neural network
fitControl <- trainControl(method = "cv", 
                           number = 5, 
                           classProbs = TRUE, 
                           summaryFunction = twoClassSummary)

nnetGrid <-  expand.grid(size = seq(from = 1, to = 10, by = 1),
                         decay = seq(from = 0.1, to = 0.5, by = 0.1))

# set up control parameters for the GBM
fitControl.GBM <- trainControl(method = "cv", 
                               number = 5,
                               classProbs = TRUE,
                               summaryFunction = twoClassSummary)

# pre process the data....only the continuous variables
preProcValues <- preProcess(df.train, method = c("range"))
trainTransformed <- predict(preProcValues, df.train)
testTransformed <- predict(preProcValues, df.test)
#----------------------------------------------------------------------

#----------------------------------------------------------------------
# Train the models
t <- Sys.time()
nnetFit <- train(y ~ ., 
                 data = trainTransformed,
                 #data = df.train,
                 method = "nnet",
                 metric = "ROC",
                 trControl = fitControl,
                 tuneGrid = nnetGrid,
                 verbose = FALSE)

# tree-based methods generally don't need input scaling
gbmFit <- train(y ~ ., 
                #data = trainTransformed,
                data = df.train,
                method = "gbm",
                metric = "ROC",
                trControl = fitControl.GBM,
                verbose = FALSE)
Sys.time() - t
Time difference of 45.61796 mins
#-------------------------------------------------------------------------

```

The first thing one might notice is that even with a modest data set it takes a while to train the models.  This is mostly due to the neural network: 5 fold cross-validation means each model is run 5 times.  Since we have a grid of hyperparameters with 50 distinct combinations, we have 250 models that are evaluated here.  Each of those 250 models is a different neural network and neural networks are generally pretty time costly to train (the backpropogation algorithm that is generally used to find optimal weights for the network nodes is pretty loop intensive).

Before moving on the prediction, I would like to highlight a few things here:

### Preprocessing data before or during?

If I look at a few of the list objects returned by train() for the neural network I can confirm that no preprocessing took place within train() and that the training data are what I supplied.

```r
> nnetFit$preProcess
NULL
> nnetFit$trainingData
# A tibble: 87,250 x 6
   .outcome    speed   depth length hour  angle
   <fct>       <dbl>   <dbl> <fct>  <fct> <dbl>
 1 notforaging     0 0.00663 3      3         0
 2 notforaging     0 0.00663 3      4         0
 3 notforaging     0 0.00663 3      4         0
 4 notforaging     0 0.00663 3      4         0
 5 notforaging     0 0.00663 3      4         0
 6 notforaging     0 0.00663 3      4         0
 7 notforaging     0 0.00663 3      4         0
 8 notforaging     0 0.00663 3      1         0
 9 notforaging     0 0.00663 3      1         0
10 notforaging     0 0.00663 3      1         0
```

An alternative to pre-processing data before supplying that data to train() would be to include a pre-processing option within the train() method.  Here's how that would look for my problem:

```r
nnetFit2 <- train(y ~ ., 
                 data = df.train,
                 method = "nnet",
                 preProcess=c('range'),
                 metric = "ROC",
                 trControl = fitControl,
                 tuneGrid = nnetGrid,
                 verbose = FALSE)
```

Here, I feed train() the original training data and include a pre-processing option inside the method call.  Here's the thing: when I pre-processed the data before passing it to train(), I used a normalization that transformed the continuous variables to the [0,1] interval.  This was done using:

```r
preProcValues <- preProcess(df.train, method = c("range"))
trainTransformed <- predict(preProcValues, df.train)

> preProcValues
Created from 87250 samples and 6 variables

Pre-processing:
  - ignored (3)
  - re-scaling to [0, 1] (3)


```

This transformation basically left my 3 factor-type variables alone and transformed my 3 continuous variables.  

When I include the pre-processing inside of my call to train(), I don't really know what's going on but I don't love it.  It looks like all of my variables are being transformed.  Maybe this doesn't matter much, but I don't like rescaling categorical variables...it doesn't make sense to me.

```r
> nnetFit2$preProcess
Created from 87250 samples and 10 variables

Pre-processing:
  - ignored (0)
  - re-scaling to [0, 1] (10)

```

###  One-hot Encoding

When I first starting running neural network models using train() I was pretty confused about why I was feeding it 5 input variables and it was coming back to me with 10 input models.  At first, I thought the train() function was doing it's own feature selection and maybe adding interaction terms and testing them on it's own.  

While it seems that, [caret::train() does have feature selection methods](https://topepo.github.io/caret/feature-selection-overview.html), this is not what was happening in my case.  In my case, since lenght and hour-of-day are categorical factor variables, the *nnet()* method that is called by train converts the categorical variables to a set of dummy variables (one-hot encoding).  Since length has 5 levels, the first one is dropped and 4 dummy variables are created.  Hour-of-day has 4 levels so the first one is dropped and we have dummy variables for hour2, hour3, and hour4.

```r
> nnetFit$finalModel
a 10-10-1 network with 121 weights
inputs: speed depth length2 length3 length4 length5 hour2 hour3 hour4 angle 
output(s): .outcome 
options were - entropy fitting  decay=0.1
> 
```

