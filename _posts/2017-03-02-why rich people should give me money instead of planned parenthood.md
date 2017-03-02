
Economists tend to think a little differently about things. We generally process things around us through the 3-tiered filter of trade-offs, opportunity costs, and credible counterfactual scenarios.  I contend that this usually results in interesting insights into social-behavioral outcomes that would not otherwise be gleaned....but I'll freely admit that sometimes (usually when applied poorly) it produces lines of thought that are just silly.

I've been thinking about this idea off-and-on for a little while and, to be totally honest, I'm only like 75% confident it has merit.  So, by my subjective probability assessment, this post has like a 25% of being total kaka.

## Executive Summary

Before you read any further I will tell you that I have split the remainder into 2 parts.  **Part I: Background** contains a fair bit of editorializing.  I believe my arguments here are logically consistent and generally well supported by verifiable fact...but if you don't care to hear my opinions on why rich Planned Parenthood donors should start thinking a little more strategically/politically, then feel free to skip Part I.  **Part II: Some Data Mining** basically involves me using demographic data to identify places in the U.S. where there is a disconnect between expected support for Planned Parenthood and the voting record of the congressional representative.

As always, the punchline first:

My first reaction to the news item that Sheryl Sandberg had donated $1 million to Planned Parenthood (PP) was not the same glee that a lot of my fellow PP supporters had.  It was a weird sensation of, 'Oh man, these kooks that want to defund PP are going to love this.  It's exactly what they want.'

**Claim**: 

If you're rich and thinking about cutting Planned Parenthood a check for a million bucks, I think you could do more good for PP by using that money a little differently.

The subtext to my claim is this: PP gets around $500 million/yr in federal funds.  If you're rich and really believe in the mission of PP I think it is in your interest to get them votes not cash.  Because if PP is defunded, the people who want to support those services won't be on the hook for a cool mil here and there...they'll need to come up with HUNDREDS of millions EVERY YEAR. 

**Methods**: 

Basically this post focuses on demonstrating how to use demographic data from the Census Bureau to find places in U.S. where you would expect more support for Planned Parenthood than currently exists. 

The data-mining exercise I'm presenting here is motivated by three observations:

1. [Poll](http://www.politico.com/story/2017/01/poll-roe-v-wade-planned-parenthood-234270) after poll [after poll](http://www.gallup.com/poll/186188/view-planned-parenthood-favorably.aspx) after poll has shown that a clear majority of Americans support Planned Parenthood.  To be fair, support for PP has declined somewhat (I suspect this is due to the recent flood of false claims about federal money paying for abortions, selling body parts and other bullshit that's been thrown about) but is still in the 60% of all Americans range.  The problem (for PP supporters) is not numbers but location: opponents of Planned Parenthood are fewer in number but spatially distributed in such a way as to grant them over-representation in Congress.  

2. The second observation is one that popped out at me after looking at a lot of demographic data: there seem to be a lot of places (congressional districts to be precise) where the demographics would appear friendly to Planned Parenthood - lots of women and educated people (two groups that according to Quinnipiac's latest poll tend to strongly support Planned Parenthood) but where the Congressperson has voted in favor of defunding PP.

3. There are things that Planned Parenthood, as a government funded entity, cannot do.  This means that some of the 'groundgame' strategies I will mention here involve *not* giving money to PP but rather giving that money to some other entity that can work on PP's behalf. For example, PP can not/should not be able to choose where to provide services based on political strategy...and PP can not/should not be able to use federal funds to target outreach based on political strategy...but some entity called saveplannedparenthood.org (perhaps with a $1 million donation from Sherly Sandberg) can do both of these things. 

My basic contention here is that, if you are a rich supporter of Planned Parenthood, you are not up against a national trend.  You enjoy the support of a clear majority of Americans.  What you are facing is many localized pockets of opposition.  To deal with diffuse, localized opposition you need a groundgame...and a million bucks can buy you A LOT of groundgame.    

**Assumptions**
People like Sheryl Sandberg would likely prefer the positive press they get from giving money to Planned Parenthood over potentially polarizing press they would get from giving money to a political campaign.  My approach is political but is not partisan and avoids the need to go after congressional seats directly.  It involves spending money strategically to strenghten the voice of people who already support Planned Parenthood. 

**Conclusions**

My conclusions (if you can even call them that) are basically 2-fold:

1. Outreach, education, and expansion of services seems like one pretty solid way to support the future of Planned Parenthood.  This approach works on the linked theory that:

* like same sex marriage, Planned Parenthood would enjoy even more political support if more people had a personal connection to someone who has been helped by PP.
* more people would have a connection to someone helped by PP if more people were aware of available services in their area and if more areas had easier to access services.

2. straight up lobbying is also an option.  

If you like Option 1 then the first pass data mining strategy is to find places currently voting to defund that have large populations of 'likely PP users.' There is some obvious bad news here: expanding services in areas that are already voting to support federal funding for PP doesn't solve the problem of needing ~25 congressional votes to ensure the future of PP.  That means there may be some very underserved areas that get passed over for investment (under the 'most bang for the Sheryl Sandberg buck) in favor of other underserved areas in districts currently voting to defund.  I KNOW this is a totally unsavory way to look at the problem BUT it should also be remembered that putting $250k into a clinic this year doesn't help much if $500,000,000 goes away next year.

If you like Option 2 the data mining strategy is a little different.  Rather than find likely Planned Parenthood users you are looking for likely pro-PP voters.  

**Caveats**: 

My main caveat here is that there is no reason this has to be a partisan exercise.  The issue is unavoidably partisan in the sense that, in the last House vote to defund Planned Parenthood, only 2 Republicans (out of like 238) opposed defunding while only 2 Democrates (out of like 193) supported defunding...but what I want to point out immediately is that I AM NOT MAKING THIS A PARTISAN ISSUE. I'm simply suggesting that there are lot of congressional districts that, demographically, look more pro-Planned Parenthood than their congressman's voting record.  A little bit of money invested in right places can 

1. help determine which of these places really are pro-Planned Parenthood
2. help those areas find their voice.

For the purposes of this piece I don't care if all 435 House seats are red....as long as the votes on any measure to defund PP actually reflect the values of the congressional districts they represent.

## Data

I used some pretty basic and easy to obtain data for this:

1. Demographic data by Congressional District from the Census Bureau.  I used the API to pull the following data directly in R
	1. population by age and sex
	2. education level by age
	3. ethnic makeup
	4. poverty status
2. I matched the demographic data with a list of Congressmen for each district and their party affiliation. [Here is a link to the .csv file]()
3. I also used a list of results from the 2016 presidential election by congressional district. [Here is a link to the .csv file]()

### Resources

1. The R Script [plannedparenthood.R](https://github.com/aaronmams/R-spatial/tree/master/R) is used to retrive the Census Bureau Data and do the analysis
2. [A couple .csv and .txt files with names and party affiliations for members of the 114th Congress are here](https://github.com/aaronmams/R-spatial/tree/master/data)

## Background

Not long ago [it was reported that Sherly Sandberg gave about a million bucks](http://www.cnbc.com/2017/02/01/facebooks-sheryl-sandberg-donated-1-million-to-planned-parenthood.html) to Planned Parenthood.  Not too long after that [Steven Colbert got some good press for giving around $800,000 to help fund South Carolina schools](http://time.com/3850694/stephen-colbert-funds-every-teacher-requested-grant-in-south-carolina-schools/).  

While I applaud these people of financial means supporting causes they value, I don't love what is says about the world we live in: that important functions of our government are basically being made contingent on the generosity of celebrities and tech executives.  

In order to keep this to a digestable length I'm going to focus on the issue of government funding for Planned Parenthood and Sheryl Sandberg's million dollar commitment to women's health.

### A Little Soap-boxing

The main reason I'm a little lukewarm on the whole 'Sheryl Sandberg donates $1 million to Planned Parenthood' isn't because I don't think she should support causes she values (assuming she does indeed values women's health and isn't just cutting a check to lessen her own tax burden). I share Sheryl Sandberg's affinity for Planned Parenthood and will go as far as saying that I think it is exactly the type of program that the government has an import role in providing.  The reason I don't love stories about rich people supporting essential government services is that there is a relatively small (but politically powerful) group of people in our country working hard to remove the government's involvement in Planned Parenthood (and NPR/PBS, and the National Science Foundation, and the National Endowment for the Arts, etc, etc. but let's stick to Planned Parenthood for today).  I'm opposed to this 'defund Planned Parenthood' group and I think they are emboldened by stories of rich people stepping in and cutting checks.  To keep it simple, I think each time a Tea Party activist hears one of these stories they say, "See, we don't need Planned Parenthood, let the libs pay for it if they like it so much."

### The basic bargain

Last bit of editorializing I promise: 

My personal feeling on this (which you are all certainly welcome to disagree with) is that when it comes to use of public monies/tax dollars there's a basic implied civil compromise we've had for a while:

I like schools, I like investing in the future (NSF, research grants to universities, etc), I like agricultural subsidies (mostly because I dig the win-win that happens when we do shit like buy surplus milk from dairy farmers to keep them from going broke when supply shocks hit and then turn around and provide that milk at very low cost to poor kids in schools).  

I don't really like that my tax dollars are used to subsidize oil companies, or that my tax dollars are used to support religious shit like [Ken Ham's weird Young Earth Creationist Playground](http://www.kentucky.com/news/state/article73971147.html) (P.S.: yes, my tax dollars go there.  I live in donor state, KY is a 'receiver' state)  

Here's the thing, there's a bunch of people in places like Alabama's 4th Congressional District who are the exact opposite of me.  They love the shit I don't like and hate the stuff I like.  But for years I feel like we had a basic compromise: I pay my taxes, they pay their taxes, they get some stuff that they like (oil subsidies and Young Earth Creation Museums), and I get some stuff I like (good public schools, poverty reduction programs, etc).  

There is a brand of, generally Tea-Party aligned voters, now who want to trash this implied compromise I mentioned. That bothers me because I may not love my taxes supporting a military-industrial complex or religious subsidies but I've never tried to kill those things.  To be clear there have always been libertarians and libertarian leaning folks who just want to take an axe to everything.  I don't typically agree with these folks but they don't really bother me.  There have also always been people who think we spend too much or spend inefficiently on some things.  Again, I got no beef (at least no beef that can't be discussed/debated like civilized adults) with somebody who says, "the current system of funding Planned Parenthood is a bad way to provide a safety net focused on women's health issue.  We need to rethink whether federal funding for Planned Parenthood is the best way to address the issues at hand."  

## The Data Part

My basic premise as I mentioned is this: if the 'defunders' get their way, Sheryl Sandberg isn't getting a break on her taxes, she's just going to see less of her money going to the shit she likes (assuming she likes Planned Parenthood, clean air, and clean water) and more of her money going to the shit that the 'defunders' like (Bible themed amusement parks, subsidies for oil and gas companies, etc.)

The 'defunders' aren't saying, "Ok, let's give everybody their money back and you pay for the stuff you want and we'll pay for the stuff we want." They're saying, "Hey Sheryl Sandberg, we'd like to take all of your tax money to pay for our stuff and, oh-by-the-way, if you like Planned Parenthood you'll have to cut a separate check for that because fuck you, that's why."
 
At this point, I feel like I should probably abstract away from Sheryl Sandberg because I don't really know anything about her politics or the strength of her emotional connection to Planned Parenthood.  So let's suppose there is an entity (let's call it the SSLE - Sheryl Sandberg Like Entity) that:

* is rich
* really likes Planned Parenthood and, importantly, wants PP to be able to continue providing health and wellness services to females for generations to come
* has $1 million bucks burning a hole in their pocket
* wants to support the mission of Planned Parenthood honestly (i.e. not just looking to lessen their own tax burden through some charitable contributions)
* is looking for the biggest impact for their gift

My contention is pretty simple:

votes are more important than money.

Planned Parenthood gets something in the neighborhood of $500 million/year in federal funding.  If congress defunds it, rich people wanting to continue providing those services aren't going to be on the hook for $1 million bucks here and there...they're gonna have to come up 100's of MILLIONS of dollars EVERY year...all well watching their tax money get appropriated for stuff they may not like, such as spending 30 billion on dumbass border wall.

The last vote on defunding Planned Parenthood was 241 - 187 (basically a straight party line vote with Lapinski and Peterson the only 2 Democrats voting to defund and Dold and Hanna the two lone Republicans voting not to defund...and 6 note voting).  So you need to gain 27/28 votes in the House to keep this issue off the table (I want to be crystal clear that I'm not suggesting Sheryl Sandberg or a SSLE become a total partisan hack or politcal animal. The votes to protect Planned Parenthood, I believe, could be gained without 'flipping' a district...I'll explain later).  

I'm not sure picking up these votes (or at least enough to convince Paul Ryan it's not worth the fight to raise the issue) would be as hard as it sounds.  

## Idea 1: Expand/Improve services in strategic areas

This probably sounds like a 'no-duh' idea and there might be a few people saying, "obviously, your fictional SSLE gives a million bucks to Planned Parenthood and then Planned Parenthood uses that money to provide health and wellness services to women...great idea Captain Obvious." 

My proposal is a bit more nuanced than that.  I'm suggesting that, in this expansion of services and education, Planned Parenthood (or more likely an agent working on their behalf) be somewhat strategic about where money is used.  

A [2009 Gallup poll](http://www.gallup.com/poll/118931/knowing-someone-gay-lesbian-affects-views-gay-issues.aspx) found that people are more likely to support marriage equality if they know someone who is gay.  Several studies and pundits since then have attributed the rise in support for same sex marriage (to the point that it is no longer even a controversial issue) to the large and growing number of people who personally know someone who is gay. 

While not exactly related to the phenomenon above, [here is an interesting discussion of the evolution of public opinion on same sex marriage by Nate Silver](http://www.gallup.com/poll/118931/knowing-someone-gay-lesbian-affects-views-gay-issues.aspx)
https://fivethirtyeight.blogs.nytimes.com/2013/03/26/how-opinion-on-same-sex-marriage-is-changing-and-what-it-means/

It's actually more than a little unsavory to think about Planned Parenthood targeting areas for facilities upgrades, outreach services, etc. based on propensity to influence congressional votes...but I see this fact as an opportunity for our SSLE.  

Solution: the SSLE gives $1 million dollars to saveplannedparenthood.org, who uses part of the donation to commission a study of the most strategic locations for Planned Parenthood Support, then uses the rest of the money to:

1. make donations to local offices ([yes, you can apparently donate directly to local area offices](https://secure.ppaction.org/site/Donation2;jsessionid=00000000.app259a?idb=756212594&df_id=12833&12833.donation=form1&mfc_pref=T&NONCE_TOKEN=B0ABB463447CD3515DC7F2DC67142725)) or

2. establish a local office or outreach/educational center in a 'high priority' area that doesn't currently have services.

**Quick Aside**: yes, technically NARAL is a group already set up to do some of the lobby-type work I'm talking about...but they are really bad at it.  [Campaign contribution reports](https://www.opensecrets.org/outsidespending/recips.php?cmte=C70002761&cycle=2016) indicate that in 2016 NARAL spent $11,000 to oppose Bobby Jindal's presidential campaign and $100,000 to defeat Marco Rubio in the senate.  Bobby Jindal never polled above 2% in any state and Marco Rubio's re-election to the senate was pretty much foregone in 2016.   The $111,000 they spent on these two opposition campaigns was totally wasted.

### How do we prioritize areas?

Presumably a consulting/strategy firm could sus this out pretty quick for $200-300k..but if the SSLE wanted to hire me to do it here is how I might start:

[Planned Parenthood by the Numbers](https://www.plannedparenthood.org/files/9313/9611/7194/Planned_Parenthood_By_The_Numbers.pdf) reports that services recipients are overwhelmingly:

* females over the age of 20 and
* people with incomes at or below 150% of the poverty line

Since I know the last House vote on defunding basically resulted in all Republicans voting to defund I don't really need to mess around with determining who voted 'defund' versus not.  I can just look for Republican Congressional Districts with populations skewed towards the biggest statistical users of PP.

The steps are carried out below but here is what I did:

1. Got the data on party affiliation of each Congressional Rep for each district

2. pulled Census Bureau data on total population and female population by age and by Congressional District

3. pulled Census Bureau data on poverty status among females ages 25 - 44 by Congressional District.

4. Pulled some other demographics that I want to use later

5.  Merged the demographic data and the party affiliation data

6. Filtered for Republican districts and sorted by female population and female poverty status to find Republican districts with demographics fitting the 'Planned Parenthood users' demographic.

```R
#----------------------------------------------------
house2016 <- read.csv('data/Congress114_members.txt')
house2016 <- strsplit(as.character(house2016[,1]),"\\s+")

#first just take out all the list elements that have exactly 5 elements because
# these will be easy to deal with
goods <- which(lapply(house2016,function(x)length(x))==5)

good.members <- data.frame(rbindlist(lapply(goods,function(i){
  tmp <- house2016[[i]]
  cd <- unlist(strsplit(tmp[[1]],"[.]"))
  data.frame(cd,firstname=tmp[2],lastname=tmp[3],party=tmp[4],state=tmp[5])
})))

#probably have to do the others by hand
bads <- which(lapply(house2016,function(x)length(x))!=5)

#there are only 13 bad ones at this point...
r1 <- data.frame(cd=23,firstname='Debbie', lastname='Wasserman Schultz', party='D',state='FL')
r2 <- data.frame(cd=8,firstname='Chris Van', lastname='Hollen', party='D',state='MD')
r3 <- data.frame(cd=2,firstname='Ann McLane', lastname='Kuster', party='D',state='NH')
r4 <- data.frame(cd=12,firstname='Bonnie', lastname='Watson Coleman', party='D',state='NJ')
r5 <- data.frame(cd=1,firstname='Michelle', lastname='Lujan Grisham', party='D',state='NM')
r6 <- data.frame(cd=3,firstname='Ben Ray', lastname='Lujan', party='D',state='NM')
r7 <- data.frame(cd=15,firstname='Jose E', lastname='Serrano', party='D',state='NY')
r8 <- data.frame(cd=18,firstname='Sean Patrick', lastname='Maloney', party='D',state='NY')
r9 <- data.frame(cd=18,firstname='Sheila', lastname='Jackson Lee', party='D',state='TX')
r10 <- data.frame(cd=1,firstname='G.K.', lastname='Butterfield', party='D',state='NC')
r11 <- data.frame(cd=30,firstname='Eddie Bernice', lastname='Johnson', party='D',state='TX')
r12 <- data.frame(cd=3,firstname='Jamie', lastname='Herrera Beutler', party='R',state='WA')
r13 <- data.frame(cd=5,firstname='Cathy', lastname='McMorris Rodgers', party='R',state='WA')

df <- rbind(r1,r2,r3,r4,r5,r6,r7,r8,r9,r10,r11,r12,r13)

house2016 <- rbind(good.members,df)

#last step is to get the party afiliation
party.afil <- lapply(house2016$party,function(x){
  tmp <- strsplit(as.character(x),'[())]')
  if(length(unlist(tmp))==2){
    party <- as.character(unlist(tmp)[2])
  }else{
    party <- as.character(tmp)
  }
return(party)  
}) 

house2016$party <- unlist(party.afil)


#-------------------------------------------------------------------------------------
key <- 'getyourownAPIkey'
#Now get 3 simple demographics for 2012 and 2016

#1. percent female
#2. percent black
#3. percent with a college degree 
#4. age structure...let's just use percent 25 - 50

# by congressional district


# age and sex
series.males <- c('001E','002E','007E','008E','009E','010E','011E','012E','013E','014E','015E','016E')
series.females <- c('026E','031E','032E','033E','034E','035E','036E','037E','038E','039E','040E')
series <- c(series.males,series.females)

series <- paste('B01001_',series,sep="")
series.names<- c('total pop','total male','m18_19','m20','m21','m22_24','m25_29','m30_34',
                 'm35_39','m40_44','m45_49','m50_54','total female','f18_19','f20','f21','f22_24',
                 'f25_29','f30_34',
                 'f35_39','f40_44','f45_49','f50_54')


pop.fn <- function(i,yr){
  resURL <- paste('http://api.census.gov/data/',yr,
                  '/acs1?get=NAME,',
                  series[i],
                  '&for=congressional+district:*&key=',key,
                  sep="")
  ljson <- fromJSON(resURL)
  ljson <- ljson[2:length(ljson)]
  tmp <- data.frame(unlist(lapply(ljson,function(x)x[1])),
                    unlist(lapply(ljson,function(x)x[2])),
                    unlist(lapply(ljson,function(x)x[3])),
                    unlist(lapply(ljson,function(x)x[4])),
                    series.names[i])
  names(tmp) <- c('name','variable','state','congressional_district','series_name')
  return(tmp)
}

age.sex <- pop.fn(i=1,yr=2015)
names(age.sex) <- c('name','total_pop','state','congressional_district','series')
age.sex$total_female <- pop.fn(i=13,yr=2015)[,2]
age.sex$f25 <- pop.fn(i=18,yr=2015)[,2]
age.sex$f30 <- pop.fn(i=19,yr=2015)[,2]
age.sex$f35 <- pop.fn(i=20,yr=2015)[,2]
age.sex$f40 <- pop.fn(i=21,yr=2015)[,2]
age.sex$f45 <- pop.fn(i=22,yr=2015)[,2]
age.sex$f50 <- pop.fn(i=23,yr=2015)[,2]

#percent black
#2015 1 yr ACS estimate
#black population
resURL <- 'http://api.census.gov/data/2015/acs1?get=NAME,B02001_003E&for=congressional+district:*&key=???'
ljson <- fromJSON(resURL)
pct.black <- data.frame(rbindlist(lapply(ljson,function(x){
  tmp <- unlist(x)
  return(data.frame(name=tmp[1],pop.black=tmp[2],state=tmp[3],cd=tmp[4]))
})))

#total population
resURL <- 'http://api.census.gov/data/2015/acs1?get=NAME,B02001_001E&for=congressional+district:*&key=???'
ljson <- fromJSON(resURL)
total.pop <- data.frame(rbindlist(lapply(ljson,function(x){
  return(data.frame(name=unlist(x)[1],total.pop=unlist(x)[2]))
})))

pct.black <- tbl_df(pct.black) %>%
              inner_join(total.pop,by=c('name'))

#---------------------------------------------------------------------------------
#poverty percentages
resURL <- 'http://api.census.gov/data/2015/acs1?get=NAME,B17001_025E&for=congressional+district:*&key=???'
ljson <- fromJSON(resURL)
poverty.f25 <- data.frame(rbindlist(lapply(ljson,function(x){
  return(data.frame(name=unlist(x)[1],pop.poverty=unlist(x)[2]))
})))

resURL <- 'http://api.census.gov/data/2015/acs1?get=NAME,B17001_026E&for=congressional+district:*&key=???'
ljson <- fromJSON(resURL)
poverty.f35 <- data.frame(rbindlist(lapply(ljson,function(x){
  return(data.frame(name=unlist(x)[1],pop.poverty=unlist(x)[2]))
})))

pov.df <- data.frame(cbind(poverty.f25[2:nrow(poverty.f25),],poverty.f35$pop.poverty[2:nrow(poverty.f35)]))
names(pov.df) <- c('name','pov25','pov35')
#---------------------------------------------------------------------------------

#-----------------------------------------------------------------------------------
#Education Level

#set up a function for this too...just for compactness
edu.fn <- function(yr){
  resURL <- paste('http://api.census.gov/data/',yr,
                  '/acs1/subject?get=NAME,S1501_C01_006E&for=congressional+district:*&key=',key,sep="")
  ljson <- fromJSON(resURL)
  name <- unlist(lapply(ljson[2:length(ljson)],function(x)x[1]))
  pop25 <- unlist(lapply(ljson[2:length(ljson)],function(x)x[2]))
  state <- unlist(lapply(ljson[2:length(ljson)],function(x)x[3]))
  congressional_district <- unlist(lapply(ljson[2:length(ljson)],function(x)x[4]))
  df <- data.frame(name=name,pop25=pop25,state=state,congressional_district=congressional_district,
                   source=paste('ACS_1yr_',yr,sep="")) 
  
  #add in the number of people 25 and over with a bachelor's degree
  resURL <- paste('http://api.census.gov/data/',yr,
                  '/acs1/subject?get=NAME,S1501_C01_012E&for=congressional+district:*&key=',key,sep="")
  ljson <- fromJSON(resURL)
  pop25_bachelors <- unlist(lapply(ljson[2:length(ljson)],function(x)x[2]))
  
  df$pop25_bachelors <- pop25_bachelors
  return(df)
}  

edu2015 <- edu.fn(yr=2015)

#add state abbreviations to the age.sex data frame
state.codes <- read.csv('data/state_fips_codes.csv') %>% select(code,abb)
names(state.codes) <- c('state.code','abb')

names(house2016) <- c('cd','firstname','lastname','party','abb')
house2016$cd <- as.numeric(as.character(house2016$cd))
house2016$cd[is.na(house2016$cd)] <-  0 

age.sex <- age.sex %>% mutate(state.code = as.numeric(as.character(state))) %>% 
            inner_join(state.codes,by=c('state.code')) %>%
            mutate(cd=as.numeric(as.character(congressional_district))) %>%
            inner_join(house2016,by=c('abb','cd'))

#now bring in the % black
pct.black <- pct.black %>% filter(row_number() > 1) %>% 
              mutate(cd=as.numeric(as.character(cd)),
                     state.code=as.numeric(as.character(state)),
                     pop.black=as.numeric(as.character(pop.black)),
                     total.pop=as.numeric(as.character(total.pop)),
                     pct.black=pop.black/total.pop) %>%
              select(cd,state.code,pct.black)

age.sex <- age.sex %>% inner_join(pct.black,by=c('state.code','cd'))


#bring in education
edu2015 <- edu2015 %>% mutate(state.code=as.numeric(as.character(state)),
                              cd=as.numeric(as.character(congressional_district)),
                              pop25=as.numeric(as.character(pop25)),
                              pop25_bachelors=as.numeric(as.character(pop25_bachelors)),
                              pct.bachelors=pop25_bachelors/pop25) %>%
          select(state.code,cd,pct.bachelors)

age.sex <- age.sex %>% inner_join(edu2015,by=c('state.code','cd'))

#add in the female poverty
age.sex <- age.sex %>% left_join(pov.df,by=c('name')) %>%
              mutate(pov.female=as.numeric(as.character(pov25))+
                       as.numeric(as.character(pov35)),
                pct.pov.female=as.numeric(as.character(pov.female))/as.numeric(as.character(total_female)))

> age.sex %>% select(name,party,pct.pov.female,pct.female) %>% 
+    filter(party=='R') %>% arrange(-pct.pov.female,-pct.female)
                                                                name party pct.pov.female pct.female
1             Congressional District 21 (114th Congress), California     R     0.09294807 0.11997973
2                Congressional District 5 (114th Congress), Kentucky     R     0.08203590 0.13007997
3           Congressional District 3 (114th Congress), West Virginia     R     0.07256008 0.12433657
4               Congressional District 5 (114th Congress), Louisiana     R     0.07208806 0.12840255
5               Congressional District 4 (114th Congress), Louisiana     R     0.06974929 0.13060522
6             Congressional District 4 (114th Congress), Mississippi     R     0.06843843 0.13148905
7                 Congressional District 2 (114th Congress), Alabama     R     0.06394077 0.13710746
8                 Congressional District 1 (114th Congress), Georgia     R     0.06357490 0.13566131
9               Congressional District 3 (114th Congress), Louisiana     R     0.06072020 0.14015800
10                  Congressional District 5 (114th Congress), Texas     R     0.06039765 0.12990302
11             Congressional District 8 (114th Congress), California     R     0.06026953 0.12637231
12                Congressional District 8 (114th Congress), Georgia     R     0.05971084 0.12878745
13               Congressional District 12 (114th Congress), Georgia     R     0.05967383 0.13269793
14            Congressional District 10 (114th Congress), California     R     0.05909745 0.13341327
15               Congressional District 4 (114th Congress), Arkansas     R     0.05726820 0.12494924
16             Congressional District 2 (114th Congress), New Mexico     R     0.05719204 0.11793210
17                Congressional District 4 (114th Congress), Alabama     R     0.05718612 0.12638659
18               Congressional District 1 (114th Congress), Arkansas     R     0.05608055 0.12807827
19         Congressional District 8 (114th Congress), North Carolina     R     0.05586123 0.12860101
20            Congressional District 23 (114th Congress), California     R     0.05563970 0.13073820
21            Congressional District 22 (114th Congress), California     R     0.05551996 0.13542188
22               Congressional District 6 (114th Congress), Kentucky     R     0.05439300 0.13676195
23              Congressional District 1 (114th Congress), Louisiana     R     0.05365683 0.14032922
24             Congressional District 4 (114th Congress), Washington     R     0.05299454 0.12357510``` 


To address something I'm sure most people would be concerned about: It's probably not in Sheryl Sandberg's (or any other rich public figure) best interest to be perceived as a blatant partisan.  That is, let's assume our SSLE doesn't want to go around throwing money at red Congressional Districts in order to 'flip' them.  That's fine with me.

For the moment, I don't really care if Darrell Issa, Mimi Walters, or any other Republican keeps their district or not.  What I am pointing out is that there are a number of districts that, demographically, look like they should be friendly to Planned Parenthood but are currently represented by someone who has voted to defund PP.  

If I were operating under the assumption that the more people know about Planned Parenthood and, more specifically the more people know a person who has been helped by Planned Parenthood, the more likely they are to oppose defunding it, then 2 courses of action seem fruitful 

1. make sure that people in areas with a large 'PP demographic' know what Planned Parenthood services are available to them and how to access those services, and

2. if PP services are not available to populations with a large 'PP demographic' figure out how some of the SSLE gift money can be used to provide such services.


## Idea 2: Target potential supporters rather than potential users

This is a little bit of spin on the data-mining above...here, rather than ask, "who are the primary users of PP," I'm going to ask,

"who and where are the usual supporters of PP?"

The basic strategy of this analysis is to find places where demographics look friendly to PP and identify which of these places are currently represented by someone voting to defund PP.  

The empirical strategy I'm going to use is pretty simple: set up a classification model to predict whether a Congressional District is Democrate or Republican based on demographics, then look at where that model fails (generates bad predictions).   

### A Classification Tree

[Classification and Regression Trees](https://en.wikipedia.org/wiki/Decision_tree_learning) are some of the most popular 'learning' algorithms...due in no small part I'm sure to their conceptual simplicity.  

Classification trees, like logistic regression or support vector machines, attempt to classify a response variable based on an input set.  They work by recursively partitioning the data and looking to form local clusters of reasonably homogeneous observations.  One of the popular features of classification trees is that they generate local predictors.  In contrast to something like logistic regression, which generates a predictive model that is supposed to hold over the entire span of the data, classification trees try to break the data down into small groups of similar observations and can apply different models on each part.

In the following I'm going to specify a model that tries to 'learn' the party affiliation of a Congressional District's representative based on three demographic factors:

1. black population as a percent of total population
2. females 25 - 45 as a percent of total population
3. percent of total population holding a bachelor's degree.
 
```R 
# try a classification tree application just for fun...

# here we will use a classification tree to predict whether the district is R or D based 
# on demographics

library(tree)

#recod Republican districts = 1
age.sex <- age.sex %>% mutate(z = ifelse(party=='R',1,0),
                              female.votingage=as.numeric(as.character(f25))+
                                               as.numeric(as.character(f30))+
                                               as.numeric(as.character(f40))+
                                               as.numeric(as.character(f50)),
                              total_pop=as.numeric(as.character(total_pop)),
                              pct.female=female.votingage/total_pop) 


#have a quick look at some factor distributions
ggplot(age.sex,aes(x=party,y=pct.bachelors)) + geom_boxplot()
ggplot(age.sex,aes(x=party,y=pct.female)) + geom_boxplot()
```

![dist pct bachelors](/images/pct_bachelors.png)
![dist pct female](/images/pct_female.png)

```R
#a classification tree
tree.model <- tree(factor(party) ~ pct.black + pct.bachelors + pct.female, data=age.sex)
tree.model

my.prediction <- predict(tree.model, age.sex) # gives the probability for each class
head(my.prediction)

#where does the classification tree do a bad job with republican districts
post.est <- age.sex %>% select(name,z) %>% mutate(pR=my.prediction) %>%
              filter(z==1 & pR<0.5) %>% arrange(pR)


plot(tree.model)
text(tree.model)
```

![tree plot](/images/classification_tree.png)

```R
#look at number correctly classified
maxidx <- function(arr) {
  return(which(arr == max(arr)))
}
idx <- apply(my.prediction, c(1), maxidx)
prediction <- c('D', 'R')[idx]

age.sex$tree.pred <- prediction

#compare number correctly classified to what we would get from a logit model
#logit model
vote.red <- glm(z~pct.bachelors+pct.female+pct.black,data=age.sex,family=binomial(link='logit'))
age.sex$pr.R <- predict.glm(vote.red, newdata = age.sex, type = "response")

age.sex <- age.sex %>% mutate(logit.correct=ifelse(pr.R>0.5 & party=='R',1,
                                                   ifelse(pr.R<0.5 & party=='D',1,0)))

logit.bad <- nrow(age.sex) - sum(age.sex$logit.correct)
tree.bad <- nrow(tree.df) - sum(tree.df$tree.correct)

logit.bad
> 115
tree.bad
> 100
```

### Results

This is a data blog so I feel like I should take a few sentences to briefly discuss this classification model.  As mentioned above, the classification tree works by partitioning the data into small groups and 'classifying' the group according to the outcome observed in most cases within the group.

Quick Example: 

Looking at the very first data partition (pct.female < 0.14763) we can see that this results in a group  with 59 Democratic districts and 4 Republican districts.  

```R
> tree.df %>% filter(pct.female > 0.14763) %>% select(name,party,pct.female,pDem,pRepub) %>% arrange(party)
                                                         name party pct.female      pDem     pRepub
1          Congressional District 9 (114th Congress), Arizona     D  0.1497974 0.9365079 0.06349206
2       Congressional District 6 (114th Congress), California     D  0.1481894 0.9365079 0.06349206
3      Congressional District 12 (114th Congress), California     D  0.1828654 0.9365079 0.06349206
4      Congressional District 13 (114th Congress), California     D  0.1610305 0.9365079 0.06349206
5      Congressional District 17 (114th Congress), California     D  0.1538009 0.9365079 0.06349206
6      Congressional District 28 (114th Congress), California     D  0.1667583 0.9365079 0.06349206
7      Congressional District 29 (114th Congress), California     D  0.1488914 0.9365079 0.06349206
8      Congressional District 30 (114th Congress), California     D  0.1502993 0.9365079 0.06349206
9      Congressional District 34 (114th Congress), California     D  0.1604516 0.9365079 0.06349206
10     Congressional District 37 (114th Congress), California     D  0.1596265 0.9365079 0.06349206
11     Congressional District 47 (114th Congress), California     D  0.1491385 0.9365079 0.06349206
12     Congressional District 52 (114th Congress), California     D  0.1498826 0.9365079 0.06349206
13     Congressional District 53 (114th Congress), California     D  0.1518292 0.9365079 0.06349206
14        Congressional District 1 (114th Congress), Colorado     D  0.1654045 0.9365079 0.06349206
15         Congressional District 9 (114th Congress), Florida     D  0.1507522 0.9365079 0.06349206
16        Congressional District 14 (114th Congress), Florida     D  0.1523759 0.9365079 0.06349206
17         Congressional District 4 (114th Congress), Georgia     D  0.1495501 0.9365079 0.06349206
18         Congressional District 5 (114th Congress), Georgia     D  0.1653659 0.9365079 0.06349206
19        Congressional District 13 (114th Congress), Georgia     D  0.1535837 0.9365079 0.06349206
20        Congressional District 4 (114th Congress), Illinois     D  0.1526338 0.9365079 0.06349206
21        Congressional District 5 (114th Congress), Illinois     D  0.1721435 0.9365079 0.06349206
22        Congressional District 7 (114th Congress), Illinois     D  0.1686240 0.9365079 0.06349206
23         Congressional District 7 (114th Congress), Indiana     D  0.1496527 0.9365079 0.06349206
24       Congressional District 2 (114th Congress), Louisiana     D  0.1487643 0.9365079 0.06349206
25        Congressional District 3 (114th Congress), Maryland     D  0.1512806 0.9365079 0.06349206
26        Congressional District 4 (114th Congress), Maryland     D  0.1505574 0.9365079 0.06349206
27        Congressional District 7 (114th Congress), Maryland     D  0.1481562 0.9365079 0.06349206
28   Congressional District 5 (114th Congress), Massachusetts     D  0.1487462 0.9365079 0.06349206
29   Congressional District 7 (114th Congress), Massachusetts     D  0.1739919 0.9365079 0.06349206
30   Congressional District 8 (114th Congress), Massachusetts     D  0.1562549 0.9365079 0.06349206
31       Congressional District 5 (114th Congress), Minnesota     D  0.1619906 0.9365079 0.06349206
32        Congressional District 1 (114th Congress), Missouri     D  0.1513161 0.9365079 0.06349206
33      Congressional District 8 (114th Congress), New Jersey     D  0.1648129 0.9365079 0.06349206
34     Congressional District 10 (114th Congress), New Jersey     D  0.1504661 0.9365079 0.06349206
35        Congressional District 5 (114th Congress), New York     D  0.1523895 0.9365079 0.06349206
36        Congressional District 6 (114th Congress), New York     D  0.1490514 0.9365079 0.06349206
37        Congressional District 7 (114th Congress), New York     D  0.1511154 0.9365079 0.06349206
38        Congressional District 8 (114th Congress), New York     D  0.1605752 0.9365079 0.06349206
39        Congressional District 9 (114th Congress), New York     D  0.1657941 0.9365079 0.06349206
40       Congressional District 10 (114th Congress), New York     D  0.1623552 0.9365079 0.06349206
41       Congressional District 12 (114th Congress), New York     D  0.2105318 0.9365079 0.06349206
42       Congressional District 13 (114th Congress), New York     D  0.1711389 0.9365079 0.06349206
43       Congressional District 14 (114th Congress), New York     D  0.1508818 0.9365079 0.06349206
44       Congressional District 15 (114th Congress), New York     D  0.1500348 0.9365079 0.06349206
45  Congressional District 4 (114th Congress), North Carolina     D  0.1541607 0.9365079 0.06349206
46 Congressional District 12 (114th Congress), North Carolina     D  0.1546355 0.9365079 0.06349206
47            Congressional District 3 (114th Congress), Ohio     D  0.1557028 0.9365079 0.06349206
48          Congressional District 3 (114th Congress), Oregon     D  0.1505278 0.9365079 0.06349206
49    Congressional District 1 (114th Congress), Pennsylvania     D  0.1596021 0.9365079 0.06349206
50    Congressional District 2 (114th Congress), Pennsylvania     D  0.1560604 0.9365079 0.06349206
51       Congressional District 5 (114th Congress), Tennessee     D  0.1592182 0.9365079 0.06349206
52           Congressional District 9 (114th Congress), Texas     D  0.1547241 0.9365079 0.06349206
53          Congressional District 30 (114th Congress), Texas     D  0.1504158 0.9365079 0.06349206
54          Congressional District 35 (114th Congress), Texas     D  0.1477420 0.9365079 0.06349206
55        Congressional District 3 (114th Congress), Virginia     D  0.1502232 0.9365079 0.06349206
56        Congressional District 8 (114th Congress), Virginia     D  0.1682299 0.9365079 0.06349206
57      Congressional District 7 (114th Congress), Washington     D  0.1644636 0.9365079 0.06349206
58      Congressional District 9 (114th Congress), Washington     D  0.1500422 0.9365079 0.06349206
59       Congressional District 4 (114th Congress), Wisconsin     D  0.1504021 0.9365079 0.06349206
60        Congressional District 26 (114th Congress), Florida     R  0.1513825 0.9365079 0.06349206
61           Congressional District 6 (114th Congress), Texas     R  0.1489073 0.9365079 0.06349206
62           Congressional District 7 (114th Congress), Texas     R  0.1579831 0.9365079 0.06349206
63          Congressional District 24 (114th Congress), Texas     R  0.1597450 0.9365079 0.06349206
>
```

The 4 Republican districts get incorrectly classified as 'D' by the model because their nearest demographic neighbors are all blue districts.  

As an additional idiot check I'm going to cross reference the list of Red Districts predicted to be Blue by looking at which districts voted for Hillary Clinton in the 2016 presidential election.

```R
> tree.df %>% filter(tree.pred=='D' & party=='R' & Dem==1) %>% arrange(-pDem)
                                                    name party  pct.black pct.bachelors pct.female Dem      pDem     pRepub
1    Congressional District 26 (114th Congress), Florida     R 0.10314315     0.1816550  0.1513825   1 0.9365079 0.06349206
2       Congressional District 7 (114th Congress), Texas     R 0.12180701     0.2870413  0.1579831   1 0.9365079 0.06349206
3 Congressional District 10 (114th Congress), California     R 0.03222968     0.1229981  0.1334133   1 0.5952381 0.40476190
4 Congressional District 39 (114th Congress), California     R 0.01741362     0.2778712  0.1378842   1 0.5952381 0.40476190
5 Congressional District 45 (114th Congress), California     R 0.02265958     0.3328972  0.1421036   1 0.5952381 0.40476190
6 Congressional District 48 (114th Congress), California     R 0.01286595     0.2813287  0.1409260   1 0.5952381 0.40476190
7  Congressional District 8 (114th Congress), Washington     R 0.03023440     0.2217260  0.1412942   1 0.5952381 0.40476190
```  

Here the outcome of the 2016 presidential election is serving as another proxy for pro-life v. pro-choice views.  I feel like this is pretty uncontroversial but if you take issue with me labeling Clinton the pro-Planned Parenthood candidate and Trump the anti-Planned Parenthood candidate [then feel free to read these quotes from the two regarding PP](https://ballotpedia.org/2016_presidential_candidates_on_abortion). 
 
What I can see from this list is that my strategy of looking at districts where the demographic predictors didn't work particularly well appears to have resulted in a small list of really interesting districts.  That is, the algorithm identified 7 districts where Congressional Representatives have voted to defund planned parenthood...even thought the population appears it should be pro-PP.  I'll also add pretty much all of these except TX-7 have [Cook's PVI scores](https://en.wikipedia.org/wiki/Cook_Partisan_Voting_Index) pretty close to even.

The next layer to look at is the remainder of Red congressional districts that the classification tree got wrong:

```R
> tree.df %>% filter(tree.correct==0 & party=='R' & Dem==0) %>% arrange(-pDem)
                                                         name party  pct.black pct.bachelors pct.female Dem      pDem     pRepub
1            Congressional District 6 (114th Congress), Texas     R 0.20656690     0.2087134  0.1489073   0 0.9365079 0.06349206
2           Congressional District 24 (114th Congress), Texas     R 0.10413504     0.2830036  0.1597450   0 0.9365079 0.06349206
3  Congressional District (at Large) (114th Congress), Alaska     R 0.03544808     0.1871453  0.1347748   0 0.5952381 0.40476190
4          Congressional District 6 (114th Congress), Arizona     R 0.03194685     0.2620192  0.1359255   0 0.5952381 0.40476190
5      Congressional District 22 (114th Congress), California     R 0.03417989     0.1585349  0.1354219   0 0.5952381 0.40476190
6      Congressional District 42 (114th Congress), California     R 0.05250239     0.1556877  0.1372551   0 0.5952381 0.40476190
7        Congressional District 14 (114th Congress), Illinois     R 0.03384786     0.2503428  0.1332169   0 0.5952381 0.40476190
8             Congressional District 3 (114th Congress), Iowa     R 0.04151815     0.2197650  0.1349475   0 0.5952381 0.40476190
9         Congressional District 4 (114th Congress), Kentucky     R 0.03516295     0.1656608  0.1324007   0 0.5952381 0.40476190
10       Congressional District 2 (114th Congress), Minnesota     R 0.04092923     0.2682256  0.1398517   0 0.5952381 0.40476190
11       Congressional District 6 (114th Congress), Minnesota     R 0.02430447     0.2052419  0.1328726   0 0.5952381 0.40476190
12           Congressional District 15 (114th Congress), Ohio     R 0.04062791     0.1913807  0.1357742   0 0.5952381 0.40476190
13          Congressional District 21 (114th Congress), Texas     R 0.04087797     0.2885222  0.1463917   0 0.5952381 0.40476190
14            Congressional District 4 (114th Congress), Utah     R 0.01762829     0.1958381  0.1366733   0 0.5952381 0.40476190
```

This one is kind of a mash-up of pretty even districts like IA-3 (Cook's PVI of 'Even') and MN-2 (Cook's PVI of R+1) and pretty solidly red districts like TX-24 (Cook's PVI of R+13) and KY-4 (Cook's PVI of R+15).

### Discussion

Here is a quick recap of what I did and why.

The what:

1. Did some basic data filtering to find congressional districts that have both a large 'likely PP user' population AND a congressional representative currently voting to defund Planned Parenthood.

2. Specified a classification model that groups congressional district's party affiliation based on demographics.  Here I used three demographic indicators commonly associated with support for Planned Parenthood: females ages 25 - 44 as a percent of total population, number of people with a bachelor's degree as a percent of total population, and black population as a percent of total population. 

The why:

Recall that the overriding motivation for this empirical exercise was to say, "if one wanted to give PP a million bucks worth of votes rather than cash where should one start looking for those votes?"

In the case of the first item above (the 'identifying likely PP service users'), the why is pretty straightforward.  This filter (which districts have relatively large female poverty rates and relatively high female population percentages) is basically identifying areas that would be expected to be negatively impacted by the disappearance of PP but where the congressional rep is actively working to bring about that end.  Some of these places (KY-5, WV-3) look to be in pretty big anti-Planned Parenthood hotbeds..but some others (CA-21, CA-10, NM-2) look like places where a little outreach, education, and a little pressure might be able to bring the congressional vote on PP in line with the needs of the population.   
  
In the case of the 2nd item above the why is all about figuring out what is going on in these districts and where money can have some leverage.  The classification model identified congressional districts that, based on demographics, would be expected to support Planned Parenthood but where the congressional rep is voting against PP.  There are a few reasons why we would observe such an outcome:

* the model is bad: this is totally plausible.  I put this together in less than a day.  If one were doing it for real one would probably want to do some model selection, testing, etc.
* outliers: TX-6 (a suburban Dallas district) comes to mind here: high educated, large female population but not very hard for me to believe that the district really is ardently anti-Planned Parenthood.
* a disconnect between the resident population and the voting population     
* party leverage vs. voter leverage

It's the last 2 cases that are the most interesting.  For sure, some of the names on my list are just outliers but some are places where congressional reps either:

* don't have a reason to listen to the Planned Parenthood supporters (perhaps because they aren't registered to vote or tend to vote in low numbers), or
* are responding to party influence over the values of the majority of the constituency

A little money coming into the district can help voting Planned Parenthood supporters hold their reps' feet to the fire and can help bring the values of the population inline with the values of the voting population.

### A quick aside on leverage

The thing about having money is that it gives you leverage.  In the 2016 election [Issa outspent Applegate 3:1](https://ballotpedia.org/California%27s_49th_Congressional_District_election,_2016) and still only beat him by less than 1 percentage point.  In CA-10, Jeff Denham spent $4 million to his opponent's $1.5 million and won by only about 3 percentage points.  If these guys are voting to defund Planned Parenthood out of party loyalty rather than as a reflection of their constituants values (presumably because they think voting the party line on PP will come back around when they wants votes on their issue) the threat of a few hundred thousand bucks injected into their district might convince them to start voting the interests of their constituency. 

### My Proposal

If I were in charge of spending ~$1 million to develop a Planned Parenthood strategy here's my top-of-the-head idea:

Start with the 21 districts I identified as 'predicted pro-PP' then,

1. Look at voter registration statistics from the various state level Secretary of State websites ([here are the stats by county for CA](http://www.sos.ca.gov/elections/report-registration/124day-gen-16/),[here are the stata by county for CO](http://www.sos.state.co.us/pubs/elections/VoterRegNumbers/VoterRegNumbers.html)).  Could part of the 'problem' be coming from low voter turnout or low voter registration numbers? *Cost estimate*: $3,000 (~20 hours of research time)   
	1. Are there lot of young people who might view Planned Parenthood favorably who are not registered to vote?
	2. Are there lots of young people, females, poor people who don't vote?	
2. Prioritize areas where there isn't an obvious explanation for the PP opposition and commission some surveys/interviews to find out where people in the district stand on funding PP. *Cost estimate*: $100,000 ($10k each to the 10 most 'promising' districts).
3. On the basis of 1 and 2 above develop a more refined strategy for allocating remaining money:
	1. among districts, and
	2. among functions (enhancing services to better serve a need population, voter registration, education/outreach, organization/mobilization of existing supporters).  
   
## Final Words

I've rambled long enough and I think you guys get the gist so let me wrap this up: something like 60% of Americans support Planned Parenthood yet PP faces extinction because a relatively small group of people weilding a disproportionately large amount of political influence really, really, really don't like PP.  

If supporters of Planned Parenthood want to make a difference they need to recognize that this isn't a national fight, it's a hyper-regional fight.  

I've presented the skeleton of a strategy that I contend could pick up 10 - 12 House votes for Planned Parenthood for less than $1 million.  I think that's a pretty decent ROI.

The best part about my plan is that it doesn't involve influence peddling, buying votes, etc. It's simply about identifying places where people already support Planned Parenthood and helping those places establish their leverage.

My strategy doesn't involve buying anyone's vote or changing anyone's mind.  Under my plan if you find out that a particular district that might have been algorithmically identified as pro-PP is really, truly anti-PP you walk away.     
