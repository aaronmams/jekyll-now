
Yes, I'm still talking about pipelines...but in my defense I think we are starting to get to the really cool stuff.

## Overview

I have two things to report on today:

1. the use of the feature selection tool [SelectKBest](http://scikit-learn.org/stable/modules/generated/sklearn.feature_selection.SelectKBest.html) from the sklearn library to narrow down a huge list of possible input variables in a prediction problem to just the most important ones.
2. the use of the [pipeline](http://scikit-learn.org/stable/modules/generated/sklearn.pipeline.Pipeline.html) tool to organize an end-to-end analysis.

## Resources

I think I've provided some decent motivation for wanting to learn more about end-to-end data pipelines 

[Machine Learning Pipelines Part 1](https://aaronmams.github.io/Machine-learning-pipelines-part-1/)

and 

[Machine Learning Pipelines Part 2](https://aaronmams.github.io/Data-Pipelines-2-data-transformations/)

I have also found the following resources pretty valuable in trying to use the pipeline() tool to set up my own data pipeline:

[Workflows in Python](https://www.civisanalytics.com/blog/workflows-in-python-using-pipeline-and-gridsearchcv-for-more-compact-and-comprehensive-code/).  Note: I suspect there are some typos in here but I'm certainly not going to hold that against the authors.

[This thing from something called Elite Data Science](https://elitedatascience.com/python-machine-learning-tutorial-scikit-learn)

## Data Setup
I'm going to use some data on home prices.  The data come from a Kaggle competition and can be acquired [here](https://www.kaggle.com/c/house-prices-advanced-regression-techniques/data).  The data are conveniently already split into a training set and testing set with around 1,500 observations in each set on about 80 different variables.  I'll probably be doing a lot with these data in the future...for now, I'll keep it simple.

```python
import pandas as pd
import numpy as np
from sklearn.feature_selection import SelectKBest, f_classif
from sklearn import linear_model
from sklearn.pipeline import Pipeline
from sklearn.metrics import mean_squared_error, r2_score
import matplotlib.pyplot as plt

train = pd.read_csv('/Users/aaronmamula/Documents/Python Projects/machine_learning/homeprices/train.csv')
test = pd.read_csv('/Users/aaronmamula/Documents/Python Projects/machine_learning/homeprices/test.csv')
testprices = pd.read_csv('/Users/aaronmamula/Documents/Python Projects/machine_learning/homeprices/sample_submission.csv')
testprices = testprices.values

test['SalePrice'] = testprices[:,1]

```

## Some Elementary Data Manipulation

I want to make things kind of easy for this first foray into actual usage of the pipeline() function.  I'm going to cut the data down to 9 numeric input columns and the target column 'SalePrice.'  I'm also going to define training inputs, testing inputs, training outputs, and testing outputs as arrays. 

```python
cols = ["LotFrontage","LotArea","YrSold","MoSold","YearBuilt","1stFlrSF","2ndFlrSF",
          "FullBath","BedroomAbvGr","SalePrice"]

#----------------------------------------------------------------------------
train = pd.DataFrame(train,columns=cols)
train = train.dropna()

array = train.values

X_train = array[:,0:9]
Y_train = array[:,9]
#---------------------------------------------------------------------------

#----------------------------------------------------------------------------
test = pd.DataFrame(test,columns=cols)
test = test.dropna()

array = test.values

X_test = array[:,0:9]
Y_test = array[:,9]
#---------------------------------------------------------------------------

```

## Automated Feature Selection

First let's check out how the Feature Selection function *SelectkBest()* works.

```python
selector = SelectKBest(f_classif, k=5)
selector.fit(X_train, Y_train)

scores = -np.log10(selector.pvalues_)
scores
Out[20]: 
array([  1.84991677,  63.3943831 ,   0.75033511,   0.32531558,
        24.19745245,  21.19348553,   9.23140002,  32.09646814,   3.73983301])

selector.get_support()

Out[26]: array([False,  True, False, False,  True,  True,  True,  True, False], dtype=bool)

```

By default the *SelectKBest()* function uses ANOVA to select the best inputs according to explained variation in the target variable.  In this case, I asked for the 5 best input variables out of the 9 available variables.  The *SelectKBest()* function scores these and returns teh 5 best.  

In this case we get 

* LotArea
* YearBuilt
* 1stFlrSF
* 2ndFlrSF
* FullBath

## The Main Event

Now, for my pipeline example I'm going to do something pretty simple:

1. transform the data using *SelectKBest()*
2. Use the resulting K-best features in a basic linear regression to explain home prices

```python
#-------------------------------------------------------------------------
#Try a simple pipeline example
selector = SelectKBest(f_classif, k=8)
#select = sklearn.feature_selection.SelectKBest(k=5)
#clf = sklearn.ensemble.RandomForestClassifier()

lreg = linear_model.LinearRegression()
​
pipe = Pipeline(steps=[('feature_selection', selector), ('lreg', lreg)])

​
pipe.fit( X_train, Y_train )
y_prediction = pipe.predict( X_test )
y_prediction
Out[33]: 
array([ 119584.91858315,  157730.77006527,  208967.01213802, ...,
        138686.15508959,  139588.07775651,  241056.27754989])

pipe.named_steps['feature_selection'].get_support()
Out[32]: array([ True,  True,  True, False,  True,  True,  True,  True,  True], dtype=bool)
#-----------------------------------------------------------------------

```

Now, the really cool stuff with pipelines comes when you start iterating over hyper-parameters.  As a really simple example, we passed the argument k=5 to the feature selection function *SelectKBest()*.  Suppose we want to:

* run the model with the 2 best, 6 best, and 8 best features from the input set
* choose the best model
* use the best model to generate a prediction 

We can do that using the [http://scikit-learn.org/stable/modules/grid_search.html](sklearn.grid_search.GridSearch).

```python
import sklearn.grid_search


parameters = dict(feature_selection__k=[2, 6, 8])

cv = sklearn.grid_search.GridSearchCV(pipe, param_grid=parameters)

cv.fit(X_train, Y_train)
cv.fit
Out[42]: 
<bound method GridSearchCV.fit of GridSearchCV(cv=None, error_score='raise',
       estimator=Pipeline(steps=[('feature_selection', SelectKBest(k=8, score_func=<function f_classif at 0x114d332a8>)), ('lreg', LinearRegression(copy_X=True, fit_intercept=True, n_jobs=1, normalize=False))]),
       fit_params={}, iid=True, n_jobs=1,
       param_grid={'feature_selection__k': [5, 8]},
       pre_dispatch='2*n_jobs', refit=True, scoring=None, verbose=0)>

y_predictions = cv.predict(X_test)
report = sklearn.metrics.mean_squared_error( Y_test, y_predictions )

report
Out[44]: 4145491943.6340261

```

Obviously, this is just the tip of the iceburg here.  We could build a really sophisticated pipeline that includes hyper-parameters related to featuere selection, model tuning parameters, etc.

It's a little unsatisfying that, in this example, we don't get a full report of all the models considered.  But keep in mind, a big selling point of data pipelines in Python is to automate the entire data transformation-model selection-model estimation-classification/prediction process.
