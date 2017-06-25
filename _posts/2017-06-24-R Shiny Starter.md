
The [Shiny](https://shiny.rstudio.com/tutorial/) package for R provides a really easy and cool way to make interactive apps. I've seen a lot of really cool sophisticated data analytics apps built with Shiny.  There are a lot of cool examples that you can check out in the app gallery [here](https://shiny.rstudio.com/gallery/).  Another set of Shiny apps I recommend checking out are bundled in a project called the [Fisheries Economics Explorer](https://dataexplorer.northwestscience.fisheries.noaa.gov/fisheye/).   

## Pre-reqs

1. get data file called gf_monthly.RDA from my github repository
2. make sure you have the 'shiny' package installed and loaded in R

## Game plan

My overarching goal for this simple proof-of-concept app is to have a process that:

1. allows a user to select from different models, model classes, or modeling assumptions available for analysis of a time-series 
2. apply the users' selections to the time-series and output a visual display of the analysis
3. allow the output to be reactive in the sense of allowing the users to update in real time any of the chosen inputs and have the output update accordingly.

What I have accomplished so far is just a very simple starting point.  The current app does the following:

1. loads a data set of commercial fishing trips taken by month off the U.S. West Coast from Jan-1994 to Aug-2016

2. allows the user to choose either a Kalman Filter-based state space model or a simple linear regression with annual and seasonal (monthly) fixed effects

3. outputs a plot of the original data and the in sample predictions of the chosen model.  For the Kalman Filter the in-sample predictions are based on the smoothed state estimates.  For the linear model the in-sample predictions are just the fitted values given by the linear predictor:

$$\hat{y} = X' \hat{\beta}$$

What I would like to add to this starter app is:

1. Add more models to choose from

2. Add more output options....basically more diagnotics (error checking) and final outputs (out-of-sample forecasts)

3. Move the code to a webserver so the app can be accessed by anyone inside a web browser.  It looks like [somebody is maintaining an io site](https://www.shinyapps.io/) similar to github.io for this purpose but I haven't looked at it in any depth.

Here's my app:

```R
df.monthly <- readRDS("/Users/aaronmamula/Documents/R projects/ShinyApps/firstapp/gf_monthly.RDA")
df.monthly <- tbl_df(df.monthly) %>% filter(year<2016) %>%
               mutate(date=as.Date(paste(df.monthly$year,"-",df.monthly$month,"-","01",sep=""),
                                   format="%Y-%m-%d"))
library(KFAS)
library(xts)
library(mgcv)

ui <- fluidPage(
  headerPanel('Fishing Trips'),
  sidebarPanel(
    selectInput('model', 'Model', c('State Space Model','Linear Model')),
    numericInput('startyr', 'Start Year', 1996,
                 min = 1994, max = 1998)
  ),
  mainPanel(
    plotOutput('plot1')
  )
)

server <- function(input, output) {
 
  trips <- reactive({
    df.monthly %>% filter(year >= input$startyr) %>% select(date,ntrips,month,year)
  })
     

  pred <- reactive({
    fitted(KFS(fitSSM(SSModel(xts(trips()$ntrips,order.by=trips()$date) ~ 
              SSMtrend(degree = 1, Q=list(matrix(NA))) + 
              SSMseasonal(period=12, sea.type="dummy", Q = NA), H = NA),
            inits=c(0.1,0.05, 0.001),method='BFGS')$model,
                    smooth= c('state', 'mean','disturbance')))
            
  })                                       
  
  results <- reactive({ 
    r <- data.frame(y=trips()$ntrips,x1=trips()$month,x2=trips()$year)
  })
  
  lmod <- reactive ({ 
    mod1 <- predict.lm(lm(y~factor(x1)+factor(x2), data = results() ) )
  })
  
  plotdf <- reactive({
    if(input$model=='State Space Model'){
      data.frame(
        rbind(
          data.frame(date=trips()$date,ntrips=trips()$ntrips,model='observed'),
          data.frame(date=trips()$date,ntrips=pred(),model='Seasonal SSM')
        )
      )
      }else{
       data.frame(
         rbind(
           data.frame(date=trips()$date,ntrips=trips()$ntrips,model='observed'),
           data.frame(date=trips()$date,ntrips=lmod(),model='Seasonal OLS')
         )
       )
      }
    
    
    })
  
    output$plot1 <- renderPlot({
      ggplot(plotdf(),aes(x=date,y=ntrips,color=model)) + geom_line() + 
                   theme_bw() + scale_color_manual(values=c('blue','black'))
  })
    

}

shinyApp(ui = ui, server = server)


```


Here's a static image of what it looks like when I run it in an R Studio Viewer Pane:

![shinyimage](/images/shinyapp.png)

## Try it out
If you run my code above (provided you have the 'Shiny' library installed and you get the gf_monthly.RDA file off of my github repo) the app will pop up in either a local browser tab or inside R Studio (depending on how you configure things).  

You can toggle between model types (currently your only options are 'State Space Model' and 'Linear Model') and set a few different starting years that can define the range of the data you want to display.

I really like the option to run the app in an interactive viewer pane inside R Studio so I'll take a quick second and show you how to do that:

1. make sure the file containing your shiny code is saved in a file called "app.R".  As long as the file has that exact name, R Studio will recognize it as a Shiny app and will give you options for how to display it.

2. in the image below notice that R Studio has recognized the code as a Shiny app and provided an options bar at the top right portion of the scripting window.  That options bar will allow to chose whether to run the app in an R Studio viewer pane, publish it to an R Shiny Server, or run it in a local browser window.

![shinyapp1](/images/shinyapp2.png)

## Motivation


## A little backstory

Several months ago I met a guy named Arvind who showed me some very cool interactive data displays that he was using to communicate model results to his data science team and higher-level decision makers at his company.  I don't remember much detail about the models or even the modeling context but I understood the main point to be that there were models generating forecasts of some sort.  
