Some Fun Data on Regional Economic Activity
================
Aaron Mamula
4/22/2021

# 

## Intro + Background

This is going to be a quick look at two kind of cool data sets that seem
really interesting and useful for evaluating the economic impacts of
Covid. I haven’t quite figured out exactly how to best use them in this
capacity yet…but I wanted to give you all a quick look at them. The data
I’m talking about are:

1.  California Taxable Sales from the California Department of Tax and
    Fee Administration

2.  Paycheck Protection Program data from the U.S. Small Business
    Administration

I plan to keep this post pretty simple. Mainly, I just want to explore
these data sets a little. I’m going to filter out a lot of the
complexity and just focus in on what the two data sets have to say about
the Retail and Food/Beverage Service Sector of the Santa Cruz County
(CA) local economy. Here’s a quick outline for the rest of this post:

  - First, I collect the PPP data for all businesses operating in Santa
    Cruz County, CA
  - I collect the California Taxable Sales by City data.
  - I filter the PPP data to include businesses defined by NAICs Codes
    associated with Retail and Food Service Establishments
  - I filter the California Taxable Sales data to include just
    observations from Santa Cruz County
  - Then, I plot the California Taxable Sales data for Santa Cruz County
    from 2009 - 2020 which provides a quick and dirty look at how
    Covid-19 impacted Retail and Food Services Transactions in Santa
    Cruz County
  - Finally, I plot the PPP data for Retail and Food Services
    businesses. This gives a quick look at how much “bailout money” was
    given to Retail and Food Services firm in Santa Cruz County. This is
    a rough approximation of how much business lost because of the
    Covid-19 pandemic was “replaced” by government money.

### Dependencies

Here are the R libraries that this module depends on:

``` r
library(dplyr)
library(ggplot2)
library(DT)
library(ggthemes)
library(zoo)
library(scales)
library(lubridate)
library(formattable)
```

There are also some data dependencies but I will discuss those in the
next section.

## Data

### Source Data

I have two primary sources of data for this Vignette:

1.  [Paycheck Protection Program Data from the Small Business
    Administration](https://data.sba.gov/dataset/ppp-foia)

2.  [California Taxable Sales by
    City](https://cdtfa.ca.gov/dataportal/api/#)

### About the Data

These data are pretty interesting and folks interested in using them for
things more serious than recreational data mining should do their own
due diligence. Here’s a quick summary:

  - The PPP program was created to help businesses retain and pay
    workers during the Covid-19 pandemic. It was a form of economic
    stimulus that was offered to businesses on the condition that at
    least 75% of the payment amount go directly to payroll. The PPP data
    available from the SBA contain around 50 different fields that
    provide information on the nature of these payments. Each row in
    each .csv is a payment. The columns contain information like name
    and address of the business, amount of PPP money received, and other
    helpful stuff (like NAICS Code for each recipient).

  - The California Taxable Sales Data that I am using are reported
    quarterly for each major city in California. The values of most
    interest are reported in two columns:

<!-- end list -->

1.  Retail and Food Services Transactions, and
2.  All Outlets Transactions

### Data Access

The PPP data are available from the SBA as a series of .csv files. I
have downloaded these, saved them locally, and created some R functions
to:

1.  read each .csv into the workspace
2.  bind the data objects together into a single data frame
3.  clean the data frame a little (mostly standardizing string fields
    for convenience)

I’ll try to crank out a follow-up post to this one with more
reproducible content on working with these PPP data.

``` r
# this part is a little cumbersome...in my PPP-EIDL Project I've created a linked set of functions to make this look nice...
#   but it's hard to run these from within my GitHub Blog so we're going old-skool here:
  public150plus <- read.csv("C:/Users/aaron.mamula/Desktop/R-Projects/PPP-EIDL-Analysis/source-data/02-01-21 Paycheck Protection Program Data/public_150k_plus.csv",
                            stringsAsFactors = F)
  public_up_to_150k_1 <- read.csv("C:/Users/aaron.mamula/Desktop/R-Projects/PPP-EIDL-Analysis/source-data/02-01-21 Paycheck Protection Program Data/public_up_to_150k_1.csv",
                                  stringsAsFactors = F)
  public_up_to_150k_2 <- read.csv("C:/Users/aaron.mamula/Desktop/R-Projects/PPP-EIDL-Analysis/source-data/02-01-21 Paycheck Protection Program Data/public_up_to_150k_2.csv",
                                  stringsAsFactors = F)
  public_up_to_150k_3<- read.csv("C:/Users/aaron.mamula/Desktop/R-Projects/PPP-EIDL-Analysis/source-data/02-01-21 Paycheck Protection Program Data/public_up_to_150k_3.csv",
                                 stringsAsFactors = F)
  public_up_to_150k_4 <- read.csv("C:/Users/aaron.mamula/Desktop/R-Projects/PPP-EIDL-Analysis/source-data/02-01-21 Paycheck Protection Program Data/public_up_to_150k_4.csv",
                                  stringsAsFactors = F)
  public_up_to_150k_5 <- read.csv("C:/Users/aaron.mamula/Desktop/R-Projects/PPP-EIDL-Analysis/source-data/02-01-21 Paycheck Protection Program Data/public_up_to_150k_5.csv",
                                  stringsAsFactors = F)
  public_up_to_150k_6 <- read.csv("C:/Users/aaron.mamula/Desktop/R-Projects/PPP-EIDL-Analysis/source-data/02-01-21 Paycheck Protection Program Data/public_up_to_150k_6.csv",
                                  stringsAsFactors = F)
  
  
  ppp.df <- rbind(public150plus,
                  public_up_to_150k_1,
                  public_up_to_150k_2,
                  public_up_to_150k_3,
                  public_up_to_150k_4,
                  public_up_to_150k_5,
                  public_up_to_150k_6)
```

These data are quite large (mainly because they are very wide) so I’m
going to filter them to include just Santa Cruz County for this
illustration:

``` r
# the PPP data have a BorrowerCounty field and a ProjectCountyName field. These two values are generally the same but 
# can be different. I'm going to argue that, for this application, the ProjectCountyName is the more important of the two 
#  because this should be a better indicator of where the money was distributed. 

ppp.df <- ppp.df %>% filter(ProjectCountyName=="SANTA CRUZ" & BorrowerState=="CA")
```

The California Taxable Sales Data by City are accessible through an API
maintained by the California Department of Tax and Fee Information. The
GET url can be parameterized in a number of ways but for my current
purposes I just want all available data:

It’s worth noting here that the object `ca.taxable` is a list object. It
is the JSON object that is returned from my API request, which contains
some meta-data about the API request itself. I extract the values of
interest from this list object by collecting the `values` list element.

``` r
ca.taxable <- jsonlite::fromJSON("https://cdtfa.ca.gov/dataportal/api/odata/Taxable_Sales_by_City?%24count=true")
ca.taxable <- ca.taxable$value
```

### Data Exploration

I don’t want to munge around with either of these data sets too much
here. They are pretty rich and one could probably do a lot of cool stuff
with them. Here, I just want to give everyone a peek at the data with
some emphasis on stuff that’s relevant for the illustration at hand:

#### PPP Data

Here’s a quick look at some of the more interesting fields in the PPP
Data for Santa Cruz County, CA:

``` r
knitr::kable(ppp.df %>% ungroup() %>% filter(row_number()<=10) %>%
                select(BorrowerName,BorrowerAddress,BorrowerCity,BorrowerZip,InitialApprovalAmount,JobsReported,FranchiseName,
                       ServicingLenderName, NAICSCode) %>% arrange(-JobsReported))
```

| BorrowerName                 | BorrowerAddress                | BorrowerCity  | BorrowerZip | InitialApprovalAmount | JobsReported | FranchiseName | ServicingLenderName            | NAICSCode |
| :--------------------------- | :----------------------------- | :------------ | :---------- | --------------------: | -----------: | :------------ | :----------------------------- | --------: |
| THRESHOLD ENTERPRISES LTD    | 23 Janis Way                   | Scotts Valley | 95066-3506  |               5439600 |          500 |               | PNC Bank, National Association |    325411 |
| AMERI KLEEN                  | 119 BEACH ST                   | WATSONVILLE   | 95076-4557  |               3490200 |          500 |               | 1st Capital Bank               |    561720 |
| ALTA VISTA FARMS LP          | 136 MARSH LN                   | WATSONVILLE   | 95076-2224  |               3680400 |          481 |               | 1st Capital Bank               |    111334 |
| ENCOMPASS COMMUNITY SERVICES | 380 Encinal St. Suite 200      | SANTA CRUZ    | 95060-2101  |               4786545 |          450 |               | Santa Cruz County Bank         |    621112 |
| SALUD PARA LA GENTE          | 195 Aviation Way, Suite 200    | WATSONVILLE   | 95076-2053  |               5218785 |          366 |               | Santa Cruz County Bank         |    624190 |
| BAY PHOTO, INC.              | 920 DISC DR                    | SCOTTS VALLEY | 95066-4544  |               3599477 |          355 |               | Santa Cruz Community CU        |    323111 |
| S. MARTINELLI & COMPANY      | 735 Beach Street               | WATSONVILLE   | 95076       |               3894618 |          326 |               | Santa Cruz County Bank         |    311411 |
| SANTA CRUZ SEASIDE COMPANY   | 400 Beach Street               | Santa Cruz    | 95060-5416  |               6236226 |          243 |               | Santa Cruz County Bank         |    713110 |
| UNIVERSAL AUDIO, INC         | 4585 Scotts Valley Drive       | SCOTTS VALLEY | 95066-4517  |               4088800 |          201 |               | Santa Cruz County Bank         |    334310 |
| NEW TEACHER CENTER           | 1205 Pacific Avenue, Suite 301 | SANTA CRUZ    | 95060-3914  |               3071357 |          133 |               | Santa Cruz County Bank         |    923130 |

#### CA Taxable Sales

Next, let’s have a quick look at the Taxable Sales Data.

``` r
ca.taxable$RetailandFoodServicesTaxableTransactions <- currency(ca.taxable$RetailandFoodServicesTaxableTransactions,digits=0L)
ca.taxable$TotalAllOutletsTaxableTransactions <- currency(ca.taxable$TotalAllOutletsTaxableTransactions,digits=0L)

knitr::kable(ca.taxable %>% filter(County %in% c("Santa Cruz","SANTA CRUZ")) %>% filter(row_number() <= 10))
```

| CalendarYear | Quarter | QuarterMonthFrom | QuarterMonthTo | County     | City              | RetailandFoodServicesTaxableTransactions | TotalAllOutletsTaxableTransactions | DisclosureFlag | PerCapita |
| -----------: | :------ | ---------------: | -------------: | :--------- | :---------------- | ---------------------------------------: | ---------------------------------: | :------------- | --------: |
|         2009 | A       |                1 |             12 | Santa Cruz | Capitola          |                             $322,595,000 |                       $355,427,000 | NA             |        NA |
|         2009 | A       |                1 |             12 | Santa Cruz | Santa Cruz        |                             $589,761,000 |                       $718,859,000 | NA             |        NA |
|         2009 | A       |                1 |             12 | Santa Cruz | Santa Cruz County |                           $1,956,754,000 |                     $2,638,469,000 | NA             |        NA |
|         2009 | A       |                1 |             12 | Santa Cruz | Scotts Valley     |                             $117,995,000 |                       $147,933,000 | NA             |        NA |
|         2009 | A       |                1 |             12 | Santa Cruz | Watsonville       |                             $381,538,000 |                       $495,137,000 | NA             |        NA |
|         2009 | Q1      |                1 |              3 | Santa Cruz | Capitola          |                              $69,397,000 |                        $76,375,000 | NA             |        NA |
|         2009 | Q1      |                1 |              3 | Santa Cruz | Santa Cruz        |                             $128,250,000 |                       $156,001,000 | NA             |        NA |
|         2009 | Q1      |                1 |              3 | Santa Cruz | Santa Cruz County |                             $426,714,000 |                       $584,519,000 | NA             |        NA |
|         2009 | Q1      |                1 |              3 | Santa Cruz | Scotts Valley     |                              $24,822,000 |                        $32,275,000 | NA             |        NA |
|         2009 | Q1      |                1 |              3 | Santa Cruz | Watsonville       |                              $84,602,000 |                       $110,666,000 | NA             |        NA |

``` r
#datatable(ca.taxable %>% filter(County %in% c("Santa Cruz","SANTA CRUZ"))) %>% 
#  formatCurrency(c("RetailandFoodServicesTaxableTransactions","TotalAllOutletsTaxableTransactions"))
```

So there are two kind of weird things about the taxable sales data:

1.  The data are mostly quarterly…but they have an annual summary row as
    well which is identified by `Quarter = A`.

2.  The geographic unit of resolution is a little funky. There are
    observations for each major city in Santa Cruz County but there are
    also observations where the `City` field = “SANTA CRUZ COUNTY”.

These issues are pretty minor and easily dealt with.

## Food Service Analysis - County

For starters, I’m going to isolate [a set of NAICS Codes with
certian 4-digit
prefixes](https://www.naics.com/six-digit-naics/?code=72).

  - 7223, 7224, 7225 are (as far as I can tell) are the main NAICS
    prefixes associated with Food Service Establisments, Restaurants,
    and Bars.
  - 4451, 4452, 4453, and 4471 are grocery and food stores,
    beer/wine/liquor stores, and gas stations
  - 4481, 4482, 4483 - clothing stores, jewelry stores, shoe stores
  - 4511 - Sporting Goods
  - 4512 - Book Stores
  - 4522 \_ Dept Stores
  - 4531 - Florists
  - 4539 - Miscellaneous stores like pet stores, art dealers, and
    tobacco stores

I can’t say for sure if these are all the NAICS Codes that should match
up with the receipts in the `RetailandFoodServicesTaxableTransactions`
column from the `ca.taxable` data frame, but it seems like a good
approximation at least.

``` r
# extract the first 4 digits for each NAICS Code
retail <- ppp.df %>% ungroup() %>% mutate(NAICS4dig=as.numeric(substring(as.character(NAICSCode),1,4))) %>%
                filter(NAICS4dig %in% c(7223,7224,7225,4451,4452,4453,4471,4481,4482,4483,4511,4512,4522,4531,4539))
retail$InitialApprovalAmount<-currency(retail$InitialApprovalAmount,digits=0L)

knitr::kable(retail %>% select(BorrowerName,BorrowerCity,InitialApprovalAmount,JobsReported,ProjectCountyName) %>%
           arrange(-InitialApprovalAmount) %>% filter(row_number() <= 10))
```

| BorrowerName                       | BorrowerCity   | InitialApprovalAmount | JobsReported | ProjectCountyName |
| :--------------------------------- | :------------- | --------------------: | -----------: | :---------------- |
| RICHARD MICHAEL HARRISON           | SOQUEL         |            $9,112,277 |           31 | SANTA CRUZ        |
| J.J.’S SALOON AND SOCIAL CLUB, LLC | SOQUEL         |            $2,165,312 |            7 | SANTA CRUZ        |
| SEA EAGLE LP.                      | Santa Cruz     |            $1,717,410 |          164 | SANTA CRUZ        |
| CAPITOLA GAYLE’S, INC.             | CAPITOLA       |            $1,345,000 |           80 | SANTA CRUZ        |
| SEA EAGLE, LP                      | SANTA CRUZ     |            $1,160,382 |          202 | SANTA CRUZ        |
| DELUXE FOODS OF APTOS, INC.        | APTOS          |              $903,187 |           79 | SANTA CRUZ        |
| CULINARY ENTERPRISES, INC.         | CAPITOLA       |              $885,855 |          136 | SANTA CRUZ        |
| WHITING’S FOOD CONCESSIONS INC     | Santa Cruz     |              $860,850 |          100 | SANTA CRUZ        |
| FLEW THE COOP INC                  | LA SELVA BEACH |              $816,400 |          194 | SANTA CRUZ        |
| STAGNARO BROTHERS SEAFOOD INC      | Santa Cruz     |              $738,467 |          103 | SANTA CRUZ        |

Next, I’m going create a really simple visual of the time-series of
taxable sales in Santa Cruz County for just the Retail and Food Services
Transactions:

``` r
# first I need to create a date from the quarterly observations...
# I'm also going to rename these really long columns
sc.taxable <- ca.taxable %>% filter(Quarter!="A") %>% 
                  mutate(County=toupper(County),City=toupper(City)) %>%
                filter(County=="SANTA CRUZ") %>% 
                mutate(date=as.Date(paste(CalendarYear,"-",QuarterMonthFrom,"-","1",sep=""),format="%Y-%m-%d")) %>%
                mutate(Q=as.yearqtr(paste(CalendarYear,Quarter,sep="")),
                       RetailFood=RetailandFoodServicesTaxableTransactions) %>%
                select(-RetailandFoodServicesTaxableTransactions)
               


ggplot(subset(sc.taxable,City=="SANTA CRUZ COUNTY"),aes(x=Q,y=RetailFood/1000000)) +
  geom_point() + geom_line() + theme_bw() + ylab("$s (millions)") + xlab("") + scale_x_yearqtr(format = "%YQ%q") +
  theme(axis.text.x=element_text(angle=45)) +
  ggtitle("Santa Cruz County Retail & Food Service Transactions") 
```

![](2021-04-26-Santa-Cruz-County-Taxable-Sales-and-PPP-Data_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->
<br>

Even if all you know about Santa Cruz County, California is that it’s
pretty close to the ocean, it’s probably not shocking to you to see
pronounced seasonality in the quarterly Retail and Food Service
Transaction Data series. I’ll admit that I *was* somewhat surprised to
see a pretty linear upward trend in Taxable Sales in Santa Cruz that’s
been at play for the last 10 years.

If one just “eye-balls” the data, it looks like there is often a pretty
big jump in Retail and Food Service transactions between Q1 and Q2, then
progressively smaller increases between the remaining quarters. I’m not
certain on the timeline of events but I kinda think the most restrictive
elements of the County-wide shelter in place order (non-essential
businesses needing to close) went into effect around March of 2020. So
it’s not super surprising to see the Q2 2020 value pretty far below the
Q2 values for other recent years

In fact, since 2009, Q2 Retail and Food Service Transactions in Santa
Cruz County have been growing by around 3-4% per year (compared with
same quarter previous year). 2020 is the first year since 2009 that Q2
Retail and Food Service Transactions declined (again compared to same
quarter previous year).

``` r
# filter for Q2 values and clean up labels for display
q3 <- sc.taxable %>% filter(Quarter=="Q2" & City=="SANTA CRUZ COUNTY") %>% select(date,Q,County,City,RetailFood) %>%
       arrange(date) %>%
         mutate(PCT_CNG=(RetailFood-lag(RetailFood))/lag(RetailFood),
                year=year(date),
                quarter="Q2") %>% select(-date,-Q,-City)
q3$RetailFood<-currency(q3$RetailFood,digits=0L)
q3$PCT_CNG <- percent(q3$PCT_CNG)
knitr::kable(q3,
             digits=c(0,0,2,0,0),
             caption="Quarter 2 Taxable Sales and Percent Change from Previous Quarter in the Retail and Food/Beverage Sector")
```

| County     |   RetailFood | PCT\_CNG | year | quarter |
| :--------- | -----------: | -------: | ---: | :------ |
| SANTA CRUZ | $489,492,000 |       NA | 2009 | Q2      |
| SANTA CRUZ | $521,519,000 |    6.54% | 2010 | Q2      |
| SANTA CRUZ | $567,292,000 |    8.78% | 2011 | Q2      |
| SANTA CRUZ | $594,107,000 |    4.73% | 2012 | Q2      |
| SANTA CRUZ | $639,874,000 |    7.70% | 2013 | Q2      |
| SANTA CRUZ | $669,644,368 |    4.65% | 2014 | Q2      |
| SANTA CRUZ | $681,535,151 |    1.78% | 2015 | Q2      |
| SANTA CRUZ | $697,889,702 |    2.40% | 2016 | Q2      |
| SANTA CRUZ | $735,575,775 |    5.40% | 2017 | Q2      |
| SANTA CRUZ | $755,090,015 |    2.65% | 2018 | Q2      |
| SANTA CRUZ | $782,691,045 |    3.66% | 2019 | Q2      |
| SANTA CRUZ | $672,220,452 | \-14.11% | 2020 | Q2      |

Quarter 2 Taxable Sales and Percent Change from Previous Quarter in the
Retail and Food/Beverage Sector

The next step in this very simple “data story” is to aggregate the PPP
funding received by Retail and Food Service Entities in Santa Cruz
County.

``` r
retail.agg <- retail %>% group_by(NAICS4dig) %>% summarise(Jobs=sum(JobsReported,na.rm=T),
                                             Dollars=sum(InitialApprovalAmount,na.rm=T)) %>%
                arrange(-Dollars)
```

    ## `summarise()` ungrouping output (override with `.groups` argument)

``` r
# This next part is pretty cumbersome...but it's gonna make the plot look way nicer
retail.agg <- retail.agg %>% 
                mutate(BusType=ifelse(NAICS4dig %in% c(7225,7224,7223),"Food Service/Bars",
                                  ifelse(NAICS4dig %in% c(4451,4452,4453),"Grocery/Food/Liquor Store",
                                  ifelse(NAICS4dig %in% c(4481,4482,4483,4512,4531),"Retail (Clothes,Books,Shoes,etc)","Other"))))

ggplot(retail.agg,aes(x=BusType,y=Dollars/1000000)) + geom_bar(stat="identity") + 
  theme(axis.text.x=element_text(angle=45)) + ylab("$ (millions)") + xlab("") + theme_tufte()
```

![](2021-04-26-Santa-Cruz-County-Taxable-Sales-and-PPP-Data_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

So here are some related observations that we can make from a quick look
at these two data set:

  - Quarterly Taxable Sales for the Retail and Food/Beverage Services
    Sector in Santa Cruz County were down about 11% in Q2 of 2020 when
    compared against Q2 of 2019
  - In dollar terms this was a reduction of about $110 million
  - PPP payments to Santa Cruz County businesses in this sector totaled
    approximately $72.4 million.

These observations are not the result of careful analysis but rather
quick and dirty data mining. They are not made for the purposes of
supporting any conclusions about the “economic impacts of Covid on Santa
Cruz County’s Retail and Food Services Sector.” They are offered as
illustrations of the type of information that can be extracted from
joining PPP data with California Taxable Sales Data. I think it’s not
too hard to see how, with a little additional rigor, these data sets
could generate some important insights about the economic/fiscal impacts
of Covid-19 on local economies.

## Bonus Content

The California Taxable Sales Data available at the city level. In the
prior sections I just used the county aggregate but it might be fun for
some people to separate out the city level contributions. Here’s a quick
picture of how county level taxable sales for Santa Cruz County are
distributed among the various cities:

``` r
#aggretate taxable sales by year and city within Santa Cruz County...also reorder factor levels
#     so that the plot looks nicer
cities.annual <- sc.taxable %>% filter(City!="SANTA CRUZ COUNTY") %>% group_by(City,CalendarYear) %>%
              summarise(RetailFood=sum(RetailFood)) %>% arrange(CalendarYear,-RetailFood) %>%
               mutate(City=factor(City, levels=c("SCOTTS VALLEY","CAPITOLA","WATSONVILLE","SANTA CRUZ")))
```

    ## `summarise()` regrouping output by 'City' (override with `.groups` argument)

``` r
ggplot(cities.annual,aes(x=CalendarYear,y=RetailFood/1000000,fill=City)) + geom_bar(stat="identity") + 
         theme_bw() + scale_fill_viridis_d() + xlab("") + ylab("$s (millions") + 
            ggtitle("Retail and Food Service Taxable Sales in Santa Cruz County, CA")
```

![](2021-04-26-Santa-Cruz-County-Taxable-Sales-and-PPP-Data_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

We can then aggregate the PPP data by the `ProjectCity` field to get a
city-level picture of PPP money in the Retail and Food Service Sector:

``` r
# use the "retail" data frame (which was derived from the source PPP data) and aggregate by city
retail.city.agg <- retail %>% mutate(ProjectCity=toupper(ProjectCity)) %>%
            group_by(ProjectCity) %>% summarise(InitialApprovalAmount=sum(InitialApprovalAmount,na.rm=T)) %>%
             arrange(-InitialApprovalAmount)
```

    ## `summarise()` ungrouping output (override with `.groups` argument)

``` r
#order factor levels
retail.city.agg <- retail.city.agg %>% mutate(ProjectCity=factor(ProjectCity,unique(retail.city.agg$ProjectCity)))

ggplot(retail.city.agg,aes(x=ProjectCity,y=InitialApprovalAmount/1000000)) + geom_bar(stat="identity") + 
      theme_bw() +
      theme(axis.text.x=element_text(angle=90)) + ylab("$s (millions)") + xlab("")
```

![](2021-04-26-Santa-Cruz-County-Taxable-Sales-and-PPP-Data_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

The first thing to notice here is that there are bunch of “cities” in
the PPP data that don’t exist in the California Taxable Sales Data. I’m
not going to do anything about that right now…but, again, if one were
interested in using these data for more serious purposes, one would need
to decide how to map cities consistently between the two data sets.

My preliminary hypothesis is that taxable sales receipts activity in
places like Aptos, Ben Lomond, and Corralitos (places that are not
specifically represented in the CA Taxable Sales Data) are probably
embedded in the County Aggregate. That is, I’m guessing that the Santa
Cruz County Aggregate in the Taxable Sales Data is larger than the sum
of Santa Cruz, Watsonville, Capitola, and Scotts Valley..and the delta
between those two things is probably the activity in these smaller
geographies that are not listed as cities in the taxable sales data set.
