I'm only the 3 billion-th blogger to write about this but for some reason, even with the interwebs saturated with python-Oracle connection examples, this still took me pretty much the whole day to figure out.

## Objective

I access a remote Oracle Database for work.  I generally pull data from database tables and mess with them in some other stats software.  I don't write to amend tables in the database ever.  

At the moment I have two main ways I work:

1. I use the RODBC package to pass an SQL query to the server and get results pulled into R as data frames
2. I sometimes use Oracle's SQL Developer to look at tables, get field names, etc., etc.

I kind of want to be able to pull data directly from the server into Python...mostly just to see if I can do it...but also because Python seems like a better platform for building apps.

## Pre-requisites

I have an Oracle Client installed on my local machine and have set up a DSN that maps to this client.  If you haven't done this you'll need to.  I can't help you with this but if you are on a Windows set up here are some resources:

* https://tensix.com/2012/06/setting-up-an-oracle-odbc-driver-and-data-source/
* https://kb.iu.edu/d/awqf

If you have the Oracle Client set up and the tnsnames.ora file configured then getting python to connect to the database is actually pretty straightforward.

## The Method

It seems that there is a robust debate over whether one should use 

* [pyodbc](https://mkleehammer.github.io/pyodbc/) or 
* [cx_Oracle](http://cx-oracle.sourceforge.net/)

for the task at hand (get data from an Oracle database into Python).  Pyodbc looks and sounds kinda like [RODBC](https://cran.r-project.org/web/packages/RODBC/index.html) which I use a lot so I chose to focus my search there.

After about a full day's worth of reading documentation and looking at various examples I found a syntax that worked for me:

```python
import pyodbc
import pandas as pd

connection = pyodbc.connect('Driver={Oracle in OraClient11g_home1};DBQ=pacfin;Uid=uid;Pwd=pw')

#test the connection
cursor = connection.cursor()

#Example command to print the unique values of the field 'pacfin_group_gear_code' 
SQLCommand = ("SELECT distinct pacfin_group_gear_code FROM pacfin_marts.comprehensive_ft")
cursor.execute(SQLCommand)
print cursor.fetchall()

> [('HKL', ), ('TWL', ), ('TLS', ), ('POT', ), ('TWS', ), ('MSC', ), ('NET', )]

#try to get results into pandas data frame...here I'm going to do a count of the field 'FTID' by year
sql = ("SELECT pacfin_year, count(FTID) FROM pacfin_marts.comprehensive_ft group by pacfin_year")
cnn = pyodbc.connect('Driver={Oracle in OraClient11g_home1};DBQ=pacfin;Uid=amamula;Pwd=mam2pac$')
data = pd.read_sql(sql, cnn)

data
Out[30]: 
    PACFIN_YEAR  COUNT(FTID)
0        1981.0    1039369.0
1        1982.0    1018429.0
2        1983.0     841329.0
3        1984.0     704278.0
4        1985.0     806843.0
5        1986.0     872706.0
6        1987.0    1092974.0
7        1988.0    1241387.0
8        1989.0    1360107.0
9        1990.0    1239093.0
10       1991.0    1193777.0
11       1992.0    1327049.0
12       1993.0    1460154.0
13       1994.0    1000942.0
14       1995.0     987166.0
15       1996.0     959688.0
16       1997.0     946521.0
17       1998.0     780234.0
18       1999.0     711917.0
19       2000.0     543381.0
20       2001.0     526102.0
21       2002.0     470503.0
22       2003.0     448218.0
23       2004.0     448636.0
24       2005.0     417165.0
25       2006.0     405049.0
26       2007.0     419557.0
27       2008.0     432731.0
28       2009.0     469646.0
29       2010.0     375191.0
30       2011.0     444726.0
31       2012.0     442759.0
32       2013.0     463439.0
33       2014.0     442455.0
34       2015.0     426278.0
35       2016.0     297082.0

```

## The 'Secret'

[This post unlocked the secret for me](http://blog.pythonaro.com/2014/07/oracle-odbc-connection-strings-how-i.html).  It turns out that most of early problems were related to the fact that I was using 'Server=' instead of 'DBQ=' in the connection string.  This is a pretty small detail but it hung me up for a while so I'm hoping this post will maybe save a few folks an hour or so.

