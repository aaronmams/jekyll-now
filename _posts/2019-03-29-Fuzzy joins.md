Been a long time....if that intro didn't immediately make you think of Christopher Walken than I'm begging you to watch this:

[...been a long time since I had dinner with the boys](https://www.youtube.com/watch?v=UmpctrXzd1E)

Ok, let's move on.  Here's the deal, I was recently trying to join to data frames together based on a date-time range.  I found this more difficult than I thought it should be so I'm documenting my adventures here in case it helps one of you out.

## Things to know:

1. I have a data frame with specific events that are time stamped.  There are millions of these events...but for the purposes of this demo I'm going to work with a few hundred thousand.  To be less cagey, events are positions of individuals that are recorded every hour.

2. I have a second data frame with trips.  These occur less frequently than events.  Some events occur on a trip and some do not.  Some trips have events and some do not.

3.  I want to join the two data frames in such a way that any event from the events data frame that occurs between the starting and ending time of any trip in the trips data frame is joined to that trip.

## Examples:

The events data frame looks like this:

| individual | lat | lon    | time                |
|------------|-----|--------|---------------------|
| 1          | 40  | -123   | 2014-06-25 16:21:12 |
| 1          | 40  | -123.2 | 2014-06-25 17:21:00 |
| 2          | 36  | -116   | 2014-06-24 08:05:00 |


The trips data frame looks like this:

| individual | trip_id | trip_start          | trip_end            |
|------------|---------|---------------------|---------------------|
| 1          | 1104834 | 2014-06-23 00:01:00 | 2014-06-26 23:59:00 |
| 1          | 1104835 | 2014-07-01 00:01:00 | 2014-07-04 23:59:00 |
| 2          | 1104839 | 2014-01-01 00:01:00 | 2014-01-01 23:59:00 |

## Specifics

I have made test files available in a randomly chosen GitHub Repo of mine:

[Rad Teaching Stuff](https://github.com/aaronmams/rad-teaching-stuff)

There are two .csv files:

1. pos_example.csv
2. id_example.csv

Download those two files and you should be ready to rock.

## Exploration

```r
library(dplyr)

 trips <- tbl_df(read.csv('id_example.csv')) %>% select(trip_id,trip_start,trip_end,individual)
> head(trips)
  trip_id          trip_start            trip_end individual
1 1104834 2014-02-22 00:01:00 2014-02-24 23:59:00          1
2 1104835 2014-02-27 00:01:00 2014-03-01 23:59:00          1
3 1104836 2014-03-10 00:01:00 2014-03-12 23:59:00          1
4 1104837 2014-04-01 00:01:00 2014-04-02 23:59:00          1
5 1104838 2014-04-08 00:01:00 2014-04-10 23:59:00          1
6 1104839 2014-04-15 00:01:00 2014-04-16 23:59:00          1
> 
> length(unique(events$trip_id))
[1] 3211
> length(unique(events$individual))
[1] 100
```

So there are 3,211 unique trips taken by 100 unique individuals in these data.

```r
events <- tbl_df(read.csv('pos_example.csv')) %>% select(lat,lon,time,individual)
head(events)
  individual      lat       lon                time
1         52 43.97926 -124.4978 2014-06-25 16:21:12
2         21 43.47957 -124.5821 2014-06-25 16:24:00
3         70 37.78950 -122.5833 2014-06-26 08:18:00
4         78 40.80707 -124.1632 2014-06-25 13:25:00
5         30 43.34558 -124.3212 2014-06-25 13:54:00
6         87 42.50400 -124.6720 2014-06-24 05:36:00

nrow(events)
[1] 239755

```

In the events data frame there are 239,755 unique events.

Now, I want to join these data sets such that any event from the events data frame that occurs during a trip gets assigned to that trip_id.  I can (theoretically) do this with a fuzzy_inner_join() from the [fuzzyjoin package](https://cran.r-project.org/web/packages/fuzzyjoin/fuzzyjoin.pdf).

```r
library(dplyr)
library(fuzzyjoin)
library(data.table)

trips$trip_start <- as.POSIXct(trips$trip_start,format="%Y-%m-%d %H:%M:%S")
trips$trip_end <- as.POSIXct(trips$trip_end,format="%Y-%m-%d %H:%M:%S")
events$time <- as.POSIXct(events$time,format="%Y-%m-%d %H:%M:%S")

t <- Sys.time()
test <- fuzzy_inner_join(events,trips,
                         by=c('individual'='individual',
                               'time'='trip_start',
                               'time'='trip_end'),
                         match_fun=list(`==`,`>=`,`<=`))

Sys.time() - t

```
With these data of relatively modest size, the fuzzy join pretty much chokes itself to death. This one locked up my computer for about a half hour before deciding that it couldn't allocate enough memory to the process.


A faster and more feasible method for doing this join is a bit of a hack but it works pretty well:

1. expand the trips data frame to include every hour of every day that the trip was active
2. round the events data frame to the nearest hour
3. join the two data frames.

It may not be as elegant as the fuzzy_inner_join() but it works and gets me what I want...while the fuzzy_inner_join just chokes on the data.

```r
#Round trip start and trip end to nearest hour...
# this is a hack but for the current purpose it's good enough
# to match events to trips within 1 hour.
trips$trip_start <- round(trips$trip_start,units='hours')
trips$trip_end <- round(trips$trip_end,units='hours')

# write a function to expand the trips data frame to have an observation
# for every hour between the start and end times of the trip
tripsV <- unique(trips$trip_id)
tripExpand <- function(t){
  dateV <- seq(trips$trip_start[trips$trip_id==t],
               trips$trip_end[trips$trip_id==t],
               by='hour')
  data.table(individual=unique(trips$individual[trips$trip_id==t]),trip=t,hour=dateV)
}
t <- Sys.time()
trips.hr <- rbindlist(lapply(tripsV,function(t)tripExpand(t)))
events$hour <- round(events$time,units="hours")
events <- merge(events,trips.hr,by=c('individual','hour'))
Sys.time() - t

Time difference of 6.669531 secs
> head(events)
  individual                hour     X      lat       lon                time    trip
1          1 2014-02-03 00:00:00 89101 46.16719 -123.9173 2014-02-02 23:32:06 1104833
2          1 2014-02-03 01:00:00 95301 46.17176 -123.9130 2014-02-03 01:06:48 1104833
3          1 2014-02-03 01:00:00 85314 46.16721 -123.9173 2014-02-03 00:32:07 1104833
4          1 2014-02-03 02:00:00 87166 46.20375 -123.9320 2014-02-03 01:33:55 1104833
5          1 2014-02-03 03:00:00 93021 46.27332 -124.1176 2014-02-03 03:11:57 1104833
6          1 2014-02-03 03:00:00 87263 46.26094 -124.0332 2014-02-03 02:33:57 1104833
> 

```
This fast, elegant data.table solution came from [Andrew Royal on StackOverflow](https://stackoverflow.com/questions/55407040/i-want-to-understand-why-lapply-exhausts-memory-but-a-for-loop-doesnt/55409460?noredirect=1#comment97570957_55409460). 


