![noaa_twitter](images/noaa_twitter.png)

So I've just about beaten this Twitter mining thing to death but I've made a couple improvements that I think make another update worthwhile: 

1. I've built out the R Shiny App a little more to include some cool time-series analyzers for hashtag engagement
2. I've created an R-Markdown file that details the end-to-end flow of the project
3. I've organized the whole thing in a GitHub repository for your collective enjoyment

## Project Recap

I've established a link to Twitter's API using the *rtweet* package which I use to grab tweets from and about NOAA Fisheries. I've also created database tables in a SQL Server database to store these tweets.  I refresh the database once-a-week by pulling the most recent 7 days worth of tweets and pushing them to the database.  From there a run a script that updates the data called on by the Shiny App.

## The Action

The GitHub Repository with all the code for the end-to-end project is [here]('https://github.com/aaronmams/noaa-social-media).

The interactive R Shiny App is [here](https://aaronmams.shinyapps.io/twitter-explorer/).

## R Skills

1. Use [RODBC](https://cran.r-project.org/web/packages/RODBC/index.html) to set up and manage the SQL Server database using R.
2. Use [rtweet](http://rtweet.info/) to search for and download tweets according to a search criteria
3. Use [R Shiny](https://shiny.rstudio.com/) to display tweets interactively in an app
4. Use [rsconnect](https://cran.r-project.org/web/packages/rsconnect/index.html) to web deploy the app from the shinyapp.io server.

