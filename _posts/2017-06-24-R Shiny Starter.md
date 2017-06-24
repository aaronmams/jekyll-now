


My overarching goal for this simple proof-of-concept app is to have a process that:

1. allows a user to select from different models, model classes, or modeling assumptions available for analysis of a time-series 
2. apply the users' selections to the time-series and output a visual display of the analysis
3. allow the output to be reactive in the sense of allowing the users to update in real time any of the chosen inputs and have the output update accordingly.

What I have accomplished so far is just a very simple starting point.  The current app does the following:

1. loads a data set of commercial fishing trips taken by month off the U.S. West Coast from Jan-1994 to Aug-2016

2. allows the user to choose either a Kalman Filter-based state space model or a simple linear regression with annual and seasonal (monthly) fixed effects

3. outputs a plot of the original data and the in sample predictions of the chosen model.  For the Kalman Filter the in-sample predictions are based on the smoothed state estimates.  For the linear model the in-sample predictions are just the fitted values given by the linear predictor:

$$y = X' \beta$$

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

#  ui = fluidPage(
#    fluidRow(
#      column(1,
#            dataTableOutput('table')
#     )
#   )
# )

server <- function(input, output) {
 
  trips <- reactive({
    df.monthly %>% filter(year >= input$startyr) %>% select(date,ntrips,month,year)
  })
   
#  selectedData <- reactive({
#    df.monthly %>% filter(year >= input$startyr) %>% select(date,ntrips,month,year) 
#  })
  

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
    #data.frame(date=trips()$date,ntrips=trips()$ntrips,yhat=pred(),yhatnaive=150,yhat_lm=lmod())
      data.frame(
        rbind(
          data.frame(date=trips()$date,ntrips=trips()$ntrips,model='observed'),
          data.frame(date=trips()$date,ntrips=pred(),model='Seasonal SSM')
        )
      )
      }else{
      #data.frame(date=trips()$date,ntrips=trips()$ntrips,yhat=pred(),yhatnaive=150,yhat_lm=lmod())
       data.frame(
         rbind(
           data.frame(date=trips()$date,ntrips=trips()$ntrips,model='observed'),
           data.frame(date=trips()$date,ntrips=lmod(),model='Seasonal OLS')
         )
       )
      }
    
#    if(input$model=='Linear Model'){
#      data.frame(selectedData(),yhat=pred(),yhatnaive=150,yhat_lm=lmod())
#    }
    
    })
  
#    output$table <- renderDataTable(gam())

    output$plot1 <- renderPlot({
      ggplot(plotdf(),aes(x=date,y=ntrips,color=model)) + geom_line() + 
                   theme_bw() + scale_color_manual(values=c('blue','black'))
  })
    

}

shinyApp(ui = ui, server = server)


```


Here's a static image of what it looks like when I run it in an R Studio Viewer Pane:

![shinyimage](/images/shinyapp.png)
