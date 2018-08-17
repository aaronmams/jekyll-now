So I recently came across this cool little paper/project/effort:

[https://osf.io/46s2f/](https://osf.io/46s2f/)

Basically a group of researchers from kind of all over the place compiled a huge data set on issuance of red and yellow cards in first division European Football (soccer) leagues.  They gave the exact same data to 29 different research teams and asked them to evaluate two specific research questions:

1. Are soccer referees more likely to give cards to dark skin toned players, and
2. Are soccer referees from countries high in skin-tone prejudice more likely to award red cards to dark skin toned players?

This project had some exposure in the popular media when [Nate Silvers FiveThirtyEight blog](https://fivethirtyeight.com/features/science-isnt-broken/#part1) put a post up bringing attention to the fact that, even with the exact same data, the estimated effect of skin tone on probability of getting a red card ranged from 

* no effect and wide confidence intervals, to
* dark skinned players are 3 times more likely to get cards with very tight confidence intervals around the estimate.

## RDF

There is an issue publicized by [Andrew Gelman](http://andrewgelman.com/2012/11/01/researcher-degrees-of-freedom/) and others called Researcher Degrees of Freedom.  It is related to the much maligned practice of p-hacking but (IMHO) more benign.  RDF refers to the fact that researchers often must make hundreds of subjective decisions about best practices as they move through an analysis

* in an analysis of whether women tend to wear pink clothes at peak fertility, does a red count as pink? what about maroon?
* in an analysis of the effect of minimum wage increases in employment in the fast food sector, should 'fast casual' restaurants be included in the data?
* should we cluster the standard errors?
* should we standardize variables? or express them in logs? or log-changes?

People have different opinions about the pitfalls and prevalence of RDF.  My reading of Gelman's points has always been that documenting and being transparent about one's research design and analysis is really, really important...but beyond that there is no reason to think that science is broken just because replication is sometimes tricky due to RDF.

## Outline for this post

Before I get too far down the rabbit hole of talking about the project and the different empirical models used to test the research questions, let me tell you why I bring this up: I went to the project repository and got the data set that was used by all 29 research teams.  My ultimate intention is to pursue my own independent model of the research question...but I start here by trying to replicate results from one of the research teams.  

Here's how I'll proceed:

1. a little detail on the data set
2. a brief discussion of one of the analyses (one by Pope and Pope: Research Team 1).
3. results of my replication attempt.

## The data

The data can be downloaded from [here](https://osf.io/46s2f/) by anyone wishing to create an account.  I did not carefully read the terms of data download but I assume I'm not supposed to distribute these data...so I won't.

The data are organized as player-referee dyads.  Basically, the observational unit is a player-referee combination.  The noteworthy fields in the data are:

* player name
* referee identifier
* number of games in which the player and referee interacted
* number of wins, loses, and ties in the games where the player-referee interacted
* a subjective rating of the player's skine tone: scale is from 1-5 with 5 being darkest.
* number of yellow cards, red cards, and 'soft' red cards issued by the referee in each player-referee pair: a 'soft' red card occurs when a player receives two yellow cards in the same game
* static player specific fields include: player's club, league (Spain, France, England, Germany), and player's position, height, weight, and age.
* implicit bias score: this is a rating of racial prejudice for each referee's home country...I assume it is included as a proxy for underlying individual-specific racism among referees.

The data contain 146,028 observations on 2,053 distinct players, and 3,147 distinct referees.

As I said, you have to download the data on your own (I can't/won't distribute it) but here is basically what it looks like:

```R
redcards <- read.csv('redcards.csv')

> head(redcards)
    playerShort           player            club leagueCountry   birthday
1 lucas-wilchez    Lucas Wilchez   Real Zaragoza         Spain 31.08.1983
2    john-utaka       John Utaka Montpellier HSC        France 08.01.1982
3   abdon-prats  Abd\xcc_n Prats    RCD Mallorca         Spain 17.12.1992
4    pablo-mari   Pablo Mar\xcc_    RCD Mallorca         Spain 31.08.1993
5    ruben-pena         Rub̩n Pe̱a Real Valladolid         Spain 18.07.1991
6  aaron-hughes     Aaron Hughes       Fulham FC       England 08.11.1979
  height weight             position games victories ties defeats goals
1    177     72 Attacking Midfielder     1         0    0       1     0
2    179     82         Right Winger     1         0    0       1     0
3    181     79                 <NA>     1         0    1       0     0
4    191     87          Center Back     1         1    0       0     0
5    172     70     Right Midfielder     1         1    0       0     0
6    182     71          Center Back     1         0    0       1     0
  yellowCards yellowReds redCards   photoID rater1 rater2 refNum refCountry
1           0          0        0 95212.jpg   0.25   0.50      1          1
2           1          0        0  1663.jpg   0.75   0.75      2          2
3           1          0        0      <NA>     NA     NA      3          3
4           0          0        0      <NA>     NA     NA      3          3
5           0          0        0      <NA>     NA     NA      3          3
6           0          0        0  3868.jpg   0.25   0.00      4          4
  Alpha_3   meanIAT nIAT       seIAT    meanExp nExp       seExp
1     GRC 0.3263915  712 0.000564112  0.3960000  750 0.002696490
2     ZMB 0.2033747   40 0.010874894 -0.2040816   49 0.061504404
3     ESP 0.3698936 1785 0.000229490  0.5882973 1897 0.001001647
4     ESP 0.3698936 1785 0.000229490  0.5882973 1897 0.001001647
5     ESP 0.3698936 1785 0.000229490  0.5882973 1897 0.001001647
6     LUX 0.3251852  127 0.003296810  0.5384615  130 0.013752210
```


## A Logit Model

Pope and Pope take a pretty simple approach and estimate the probability of a player getting a red or yellow card in any match using a logit model and including a covariate for skin tone.

In order to do this they transform the data to make the observational unit a player-referee-game triple.  Note that, in the event that a player and referee interacted in more than 1 game, the original data don't allow us to determine, in the event of a red or yellow card, which game the card occurred in.  My data transformation is below.  Note that when I expand each player-referee pair to a player-referee-game I arbitrarily assign cards to the first rows of the grouped data.  To be more explicit: 

suppose a player-referee pair from the original data has the value '3' for the games field and the value '2' for the 'yellowCard' field, indicating that the player and referee interacted in 3 total games and the referee issued the player 2 yellow cards in those games.  I expand the data so that rather than having 1 row for this player-referee pair, there are 3 rows.  I create a 0,1 variable indicating whether or not the player got a yellow card.  For this player-referee pair I assign the first 2 rows the value 1 for the yellow card variable and assign the 3rd row the value 0.

I don't have Pope and Pope's code so I don't know exactly how they massaged the data...but here's how I did what they say they did:

```R

# create a 0,1 variable for whether the player got a yellow card...since the original data
# don't allow us to see which game (in the event of a single card but multiple games for 
# the player-referee dyad) the card occurered in, we just assign cards to games #arbitrarily.

logit.df <- test.df %>% group_by(playerShort,refNum) %>% mutate(rwnm=row_number()) %>% arrange(playerShort,refNum) %>% mutate(yc_yesno=ifelse(yellowCards<row_number(),0,1))

print.data.frame(logit.df[1:50,])

    playerShort games yellowCards yellowReds redCards rater1 rater2 refNum
1  aaron-hughes     1           0          0        0   0.25      0      4
2  aaron-hughes     1           0          0        0   0.25      0     66
3  aaron-hughes    26           0          0        0   0.25      0     77
4  aaron-hughes    26           0          0        0   0.25      0     77
5  aaron-hughes    26           0          0        0   0.25      0     77
6  aaron-hughes    26           0          0        0   0.25      0     77
7  aaron-hughes    26           0          0        0   0.25      0     77
8  aaron-hughes    26           0          0        0   0.25      0     77
9  aaron-hughes    26           0          0        0   0.25      0     77
10 aaron-hughes    26           0          0        0   0.25      0     77
11 aaron-hughes    26           0          0        0   0.25      0     77
12 aaron-hughes    26           0          0        0   0.25      0     77
13 aaron-hughes    26           0          0        0   0.25      0     77
14 aaron-hughes    26           0          0        0   0.25      0     77
15 aaron-hughes    26           0          0        0   0.25      0     77
16 aaron-hughes    26           0          0        0   0.25      0     77
17 aaron-hughes    26           0          0        0   0.25      0     77
18 aaron-hughes    26           0          0        0   0.25      0     77
19 aaron-hughes    26           0          0        0   0.25      0     77
20 aaron-hughes    26           0          0        0   0.25      0     77
21 aaron-hughes    26           0          0        0   0.25      0     77
22 aaron-hughes    26           0          0        0   0.25      0     77
23 aaron-hughes    26           0          0        0   0.25      0     77
24 aaron-hughes    26           0          0        0   0.25      0     77
25 aaron-hughes    26           0          0        0   0.25      0     77
26 aaron-hughes    26           0          0        0   0.25      0     77
27 aaron-hughes    26           0          0        0   0.25      0     77
28 aaron-hughes    26           0          0        0   0.25      0     77
29 aaron-hughes     2           0          0        0   0.25      0    163
30 aaron-hughes     2           0          0        0   0.25      0    163
31 aaron-hughes    16           2          0        0   0.25      0    194
32 aaron-hughes    16           2          0        0   0.25      0    194
33 aaron-hughes    16           2          0        0   0.25      0    194
34 aaron-hughes    16           2          0        0   0.25      0    194
35 aaron-hughes    16           2          0        0   0.25      0    194
36 aaron-hughes    16           2          0        0   0.25      0    194
37 aaron-hughes    16           2          0        0   0.25      0    194
38 aaron-hughes    16           2          0        0   0.25      0    194
39 aaron-hughes    16           2          0        0   0.25      0    194
40 aaron-hughes    16           2          0        0   0.25      0    194
41 aaron-hughes    16           2          0        0   0.25      0    194
42 aaron-hughes    16           2          0        0   0.25      0    194
43 aaron-hughes    16           2          0        0   0.25      0    194
44 aaron-hughes    16           2          0        0   0.25      0    194
45 aaron-hughes    16           2          0        0   0.25      0    194
46 aaron-hughes    16           2          0        0   0.25      0    194
47 aaron-hughes    10           2          0        0   0.25      0    233
48 aaron-hughes    10           2          0        0   0.25      0    233
49 aaron-hughes    10           2          0        0   0.25      0    233
50 aaron-hughes    10           2          0        0   0.25      0    233
   height weight      club leagueCountry    position rwnm yc_yesno tone
1     182     71 Fulham FC       England Center Back    1        0 0.25
2     182     71 Fulham FC       England Center Back    1        0 0.25
3     182     71 Fulham FC       England Center Back    1        0 0.25
4     182     71 Fulham FC       England Center Back    2        0 0.25
5     182     71 Fulham FC       England Center Back    3        0 0.25
6     182     71 Fulham FC       England Center Back    4        0 0.25
7     182     71 Fulham FC       England Center Back    5        0 0.25
8     182     71 Fulham FC       England Center Back    6        0 0.25
9     182     71 Fulham FC       England Center Back    7        0 0.25
10    182     71 Fulham FC       England Center Back    8        0 0.25
11    182     71 Fulham FC       England Center Back    9        0 0.25
12    182     71 Fulham FC       England Center Back   10        0 0.25
13    182     71 Fulham FC       England Center Back   11        0 0.25
14    182     71 Fulham FC       England Center Back   12        0 0.25
15    182     71 Fulham FC       England Center Back   13        0 0.25
16    182     71 Fulham FC       England Center Back   14        0 0.25
17    182     71 Fulham FC       England Center Back   15        0 0.25
18    182     71 Fulham FC       England Center Back   16        0 0.25
19    182     71 Fulham FC       England Center Back   17        0 0.25
20    182     71 Fulham FC       England Center Back   18        0 0.25
21    182     71 Fulham FC       England Center Back   19        0 0.25
22    182     71 Fulham FC       England Center Back   20        0 0.25
23    182     71 Fulham FC       England Center Back   21        0 0.25
24    182     71 Fulham FC       England Center Back   22        0 0.25
25    182     71 Fulham FC       England Center Back   23        0 0.25
26    182     71 Fulham FC       England Center Back   24        0 0.25
27    182     71 Fulham FC       England Center Back   25        0 0.25
28    182     71 Fulham FC       England Center Back   26        0 0.25
29    182     71 Fulham FC       England Center Back    1        0 0.25
30    182     71 Fulham FC       England Center Back    2        0 0.25
31    182     71 Fulham FC       England Center Back    1        1 0.25
32    182     71 Fulham FC       England Center Back    2        1 0.25
33    182     71 Fulham FC       England Center Back    3        0 0.25
34    182     71 Fulham FC       England Center Back    4        0 0.25
35    182     71 Fulham FC       England Center Back    5        0 0.25
36    182     71 Fulham FC       England Center Back    6        0 0.25
37    182     71 Fulham FC       England Center Back    7        0 0.25
38    182     71 Fulham FC       England Center Back    8        0 0.25
39    182     71 Fulham FC       England Center Back    9        0 0.25
40    182     71 Fulham FC       England Center Back   10        0 0.25
41    182     71 Fulham FC       England Center Back   11        0 0.25
42    182     71 Fulham FC       England Center Back   12        0 0.25
43    182     71 Fulham FC       England Center Back   13        0 0.25
44    182     71 Fulham FC       England Center Back   14        0 0.25
45    182     71 Fulham FC       England Center Back   15        0 0.25
46    182     71 Fulham FC       England Center Back   16        0 0.25
47    182     71 Fulham FC       England Center Back    1        1 0.25
48    182     71 Fulham FC       England Center Back    2        1 0.25
49    182     71 Fulham FC       England Center Back    3        0 0.25
50    182     71 Fulham FC       England Center Back    4        0 0.25

# now we'll just create the average skin-tone variable and run a logit
logit.df <- logit.df %>% ungroup() %>% rowwise() %>% mutate(tone=mean(rater1,rater2,na.rm=T))

```

## My Logit Model

Spoiler alert: my logit application here doesn't actually address the research question as stated.  When I tried to replicate Pope and Pope's logit model describing probability of getting a yellow card it took my model about 5 hours to run...I'm running some of Pope and Pope's other models now but, at this rate, those results won't be available until next week.

The important thing to note here is that from Pope and Pope, Table 8, p.8 we see the following model estimated via logistic regression:

$$ y_{i} = f(skin_tone, height, weight, age, club, position, league, referee)$$

From Table 8 we also see that the coefficient estimate on the *skin tone rating* variable was -0.00118 with a standard error of 0.005, giving a significant p-value for the estimate. 


```R
# first sample the data so we don't wait all day
players <- sample(unique(redcards$playerShort),100)

logit.df <- logit.df %>% filter(playerShort %in% players)

t <- Sys.time()
mod1 <- glm(yc_yesno~tone+height+weight+factor(position)+factor(club)+factor(refNum)+factor(leagueCountry),
+             family=binomial,data=logit.df)
Sys.time() - t
Time difference of 5.837718 hours

summary(mod1)

Call:
glm(formula = yc_yesno ~ tone + height + weight + factor(position) + 
    factor(club) + factor(refNum) + factor(leagueCountry), family = binomial, 
    data = test.df)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-1.8594  -0.5775  -0.4789  -0.3404   3.0591  

Coefficients: (3 not defined because of singularities)
                                       Estimate Std. Error z value Pr(>|z|)    
(Intercept)                           1.829e+01  2.400e+03   0.008 0.993917    
tone                                 -7.384e-02  2.960e-02  -2.495 0.012605 *  
height                               -1.270e-02  2.218e-03  -5.724 1.04e-08 ***
weight                                1.339e-02  2.089e-03   6.409 1.46e-10 ***
factor(position)Center Back           4.351e-01  3.590e-02  12.122  < 2e-16 ***
factor(position)Center Forward       -1.953e-01  3.796e-02  -5.144 2.69e-07 ***
factor(position)Center Midfielder     3.590e-01  4.657e-02   7.709 1.27e-14 ***
factor(position)Defensive Midfielder  6.157e-01  3.485e-02  17.667  < 2e-16 ***
factor(position)Goalkeeper           -1.029e+00  5.210e-02 -19.747  < 2e-16 ***
factor(position)Left Fullback         3.253e-01  3.819e-02   8.519  < 2e-16 ***
factor(position)Left Midfielder       1.067e-01  4.318e-02   2.471 0.013478 *  
factor(position)Left Winger          -6.786e-02  4.849e-02  -1.400 0.161631    
factor(position)Right Fullback        4.376e-01  3.759e-02  11.640  < 2e-16 ***
factor(position)Right Midfielder      1.336e-01  4.593e-02   2.910 0.003614 ** 
factor(position)Right Winger         -1.008e-01  4.689e-02  -2.150 0.031595 *
.
.
.

```

Note that several thousand fixed effects (club, league, referee) have been suppressed in order to avoid filling up 12 pages with regression results.

### My logit results

I don't get the same coefficient estimate as Pope and Pope (not surprising since I use a different variation of the data and I forgot to include the players age).  However, I do get basically the same qualitative result:

the estimated effect of skin tone on probability of getting a yellow card is negative and significant, suggesting no systematic racism among soccer referees.


# Final Words

## General closing words

I think the really interesting thing about this study is laid out right in the abstract:

*This investigation shows how researchers vary in their analytical approaches and makes transparent how results vary based on analytical choices. We aim to address the current lack of knowledge about just how much diversity in analytic choice exists with regard to the same data, and whethersuch diversity results in different conclusions.*


People will take away from this study different conclusions about the 'trustworthyness' of empirical work.  Rather than get myself all worked up about whether we can really 'believe' any study, I choose to focus on what I believe to be the real meat of this study: the importance of being transparent with regard to subjective analytical choices made in the investigation of important research questions.

## A specific point about my replication attemp

If you want to do this yourself I suggest you don't bother sampling the data frame.  I didn't think this part through very well.  The computational complexity here comes from trying to include several thousand different fixed effects in a logit model...not necessarily from the size of the data.
