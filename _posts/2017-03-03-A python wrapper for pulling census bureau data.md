Super short post here because the day is pretty much over...but I just discovered a Python module for grabbing Census Data and I had to share.

My [last couple posts](https://aaronmams.github.io/Automating-census-data-pulls-with-R/) have been leveraging a lot of Census Bureau data.  So far I've been working mostly in R using the API to pull data series and the R Package *RJSONIO* to clean up the JSON values returned.

## Resources
* [A github repo with examples](https://github.com/CommerceDataService/census-wrapper)
* [A slightly different option](https://stharrold.github.io/20160110-etl-census-with-python.html)

## The Action
First things first: I use the Anaconda Python Distribution and I installed the **census** module from the terminal with:

```python
conda install -c conda-forge census=0.8.1
```
if shit worked right you'll probably see something like this:

![conda screenshot](/images/conda_screenshot.png)

Use the 'get' method to grab total number of males 18-19 by state and coerce to pandas data frame.  You can see other available methods [here](https://pypi.python.org/pypi/census).

```python
from census import Census
from us import states
import pandas as pd
import pandas.io.data as web
import datetime
import matplotlib.pyplot as plt
from matplotlib import style
import numpy as np

#total males ages 18-19
pd.DataFrame(c.acs5.get('B01001_007E', {'for': 'state:*'}))

pd.DataFrame(c.acs5.get('B01001_007E', {'for': 'state:*'}))

Out[8]: 
   B01001_007E state
0        68780    01
1        11003    02
2        95106    04
3        41876    05
4       568697    06
5        73833    08
6        54373    09
7        13411    10
8        10439    11
9       250662    12
10      152668    13
11       17382    15
12       22178    16
13      185243    17
14       98126    18
15       46968    19
16       43268    20
17       60624    21
18       64135    22
19       18255    23
20       83469    24
21      103932    25
22      147839    26
23       74643    27
24       45618    28
25       85685    29
26       14510    30
27       27218    31
28       34730    32
```

The only issue I've run into so far is that the most recent year referenced in the package documentation is 2013, which makes me wonder if I'm using the most recent package version...I'll post an update if I uncover anything important on this front. 

That's it. That's all I got today.  I haven't worked with the Python census wrapper yet enough to know if I prefer Python to R for working with Census Data. I'll report back when I form a more concrete opinion on the matter.
