Had occassion to setup an SQLite database the other day. I found it be a most enjoyable experience so I thought I'd share.  In this short post I'll:

* use the Mac Terminal and sqlite shell to set up a local embedded database
* use the RSQLite package to access database tables from R.

## Background

I joined Kaggle's [Data Science for Good: Donors Choose](https://www.kaggle.com/donorschoose/io/discussion/56030) challenge. The purpose of the challenge is to develop a data analytics methodology to help the education non-profit site [Donors Choose](https://www.donorschoose.org/) increase contributions to public school classrooms.  

## Problem Statement

This first pinch-point I encountered with this challenge was that the data were made available as as series of very large .csv files.  This was making even the elementary task of running summary stats and just 'getting a feel' for the data cumbersome and time consuming.  For instance:

* one of the data files that contained info on donations to past project was about 500 MB and took about 4 minutes just to load into an R workspace using the read.csv() method. 
* the data file with information about individual projects was 2.2 GB and took almost 10 minutes to bring into R using the read.csv() method.

Note: [the data files are available for download here](https://www.kaggle.com/donorschoose/io/data)

## Solution

Since I don't really need all 4 million observations in the Donations data or all 2 million observations in the Projects in order to 'get a feel' for the data, I shoved the data files into a relational database that I could query for smaller chunks of data.

## Execution

### Step 1: create a database

I did this using the sqlite shell and I established the database in the directory that was previously set up for this R Studio Project:

```bash
cd Users/aaronmamula/Documents/R projects/kaggle/donors-choose/data
aarons-MacBook-Air-2:data aaronmamula$ sqlite3 DCTables.db
SQLite version 3.13.0 2016-05-18 10:57:30
Enter ".help" for usage hints.
```

### Step 2: load .csv files into database as DB tables

``bash
sqlite> .mode csv
sqlite> .import Projects.csv projects
sqlite> select count(*) from projects;
1208651

``
