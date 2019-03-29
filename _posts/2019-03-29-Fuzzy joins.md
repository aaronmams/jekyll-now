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

I have made test files available in a randomly chosen GitHub Repo of mine.
