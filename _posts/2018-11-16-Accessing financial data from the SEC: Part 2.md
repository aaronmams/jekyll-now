
![homer](/images/homer_donut.jpeg)

In [Part 1](https://aaronmams.github.io/Accessing-Financial-Data-from-the-SEC-Part-1/) of this series I covered a ghetto web-crawling route used to pull information from 10-K financial disclosures from the SEC's EDGAR database.  That solution relied on the R package [finreportr](https://cran.r-project.org/web/packages/finreportr/index.html) which gets points for being very easy to use but loses points for being a little unpredicatable in terms of what data it actually returns you.

The solution I'll talk about here uses the following approach:

1. Use functions from the [edgarWebR](https://github.com/mwaldstein/edgarWebR) library to get details on available 10-K filings for a company.
2. Use the [XBRL](https://cran.r-project.org/web/packages/XBRL/index.html) library to parse the .xml documents found in Step 1 above.
3. Use functions in the [finstr](https://github.com/bergant/finstr) library to further parse data retreived into objects of class 'financial statement'

# General Approach

Each 10-K filing in the SEC EDGAR database is affiliated with an index page.  This is the main url that houses key meta-data for a 10-K filing.  For example:

[https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/0001193125-17-059914-index.htm](https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/0001193125-17-059914-index.htm)

Is the index url for the annual 10-K filed by BJ's Restaurant Group on 2/28/2017.

This index page displays the links to the individual documents in the filing.  For this approach we will be looking for the XBRL INSTANCE DOCUMENT for a 10-K filing.  Once we find the url for that XBRL INSTANCE DOCUMENT we use it to parse the associated .xml file.  At that point the data has been retreived.

The final step in this approach uses the [finstr](https://github.com/bergant/finstr) library to organize the data into a structured format that can be read like a tabular financial statement.

# Desert First

A couple conclusion-y thoughts from my first two attempts at scraping SEC financial disclosures:

1. [finreportr](https://cran.r-project.org/web/packages/finreportr/index.html) is kinda cool but the package developer is really unresponsive, which leads me to believe he is not interested in improving these functions...which leads me to believe that he has found a better way to webscrap the SEC for financial disclosures.  If the package developer isn't even using the package for it's intended purpose, I think that's a strong indication we should probably not mess with *finreportr*

2. The SEC sucks and I believe they have no interest in making useful data available to the public.  If they did, they would have some manner of data standardization protocols in place and some manner of efficient distribution.  They have neither.  They suck.

3. The solution I'll talk about today: a pipeline involving [edgarWebR](https://github.com/mwaldstein/edgarWebR), [XBRL](https://cran.r-project.org/web/packages/XBRL/index.html), and [finstr](https://github.com/bergant/finstr) is (in my limited experience) a little less tempermental than *finreportr* but still has some issues that aren't really it's fault but still limit the utility somewhat.  The key limitations are:

* there has to be an XBRL INSTANCE DOCUMENT in the document library for the desired 10-K filing in order for this pipeline to work, and
* sometimes the url requests get booted off and throw errors that can break looping routines.


# Prereqs

We start with the same data frame we had last time which has a list of publicly traded restaurant groups that I want info for.  Also, not that I'm putting all necessary libraries in this code chunk.

```r
library(edgarWebR)
library(XBRL)
library(finstr)
library(dplyr)
library(ggplot2)
library(ggthemes)

stores <- data.frame(ticker=c('BJRI','BLMN','BUWR','BOBE','EAT','BUCA','BWLD','BKC', 'CPKI','CBO','CRW','TAST','CEC','CMG','CHUY','CKR','CBRL',                              'DRI','PLAY','DFRG','DENN','DIN','DPZ','DNKN','JOES.OB','BAGL',                              'LOCO','DAVE','FCCG.OB','FOGO','BAJA','HABT','KKD','LNY', 'LGNS','LUB','MSSR','MCD','MRT','NATH','NDLS','CHUX','PNRA','PFCB','PZZI','PBPB', 'RRGB','RICK','RUBO','RT','RUTH','SHAK','SWRG','SONC','SBUX','TXRH','SNS','THI','WEST','WING','YUM','ZOES'),
 name=c("BJ's",
 "Bloomin' Brands",
 "Blue Water Restaurant Group",
 "Bob Evans Farms, Inc",
 "Brinker International, Inc", 
 "Buca Inc", 
 "Buffalo Wild Wings, Inc",
 "Burger King Holdings",
 "California Pizza Kitchen",
 "Caribou Coffee Co",
 "Carlson Restaurants Worldwide",
 "Carrols Restaurant Group, Inc",
 "CEC Entertainment, Inc","Chiptole Mexican Grill",
 "Chuy's Holdings",
 "CKE Restaurants, Inc",
 "Cracker Barrel Old Country Store, Inc.",
 'Darden Restaurants, Inc',
 "Dave & Busters Entertainment, Inc",
 "Del Frisco's Restaurant Group",
 "Denny's Corp",
 "DineEquity, Inc",
 "Domino's Pizza, Inc.",
 "Dunkin Brands Group",
 "Eat At Joe's Ltd","Einstein Noah Restaurant Group, Inc",
  "El Pollo Loco","Famous Dave's of America, Inc","Fog Cutter Capital Group Inc","Fogo De Chao, Inc.",
   "Fresh Enterprises Inc","Habit Restaurants Inc","Krispy Kreme Doughnuts, Inc","Landry's Restaurants, Inc",
  "Logan's Roadhouse Inc","Luby's, Inc","McCormick & Schmick's Seafood Restaurants, Inc",
  "McDonald's Corporation","Morton's Restaurant Group Inc","Nathan's Famous, Inc","Noodles & Co",
  "O'Charley's Inc","Panera Bread Co","P.F. Chang's China Bistro, Inc","Pizza Inn, Inc","Potbelly Corp",
  "Red Robin Gourmet Burgers, Inc","Rick's Cabaret International, Inc","Rubio's Restaurants Inc",
  "Ruby Tuesday, Inc","Ruth's Hospitality Group, Inc","Shake Shack","Smith and Wollensky",
 "Sonic Corp","Starbuck Corporation","Texas Roadhouse, Inc","The Steak and Shake Company",
 "Tim Horton's, Inc", "Western Sizzlin Corporation","Wingstop Inc","Yum Brands","Zoe's Kitchen"))

```

# Quick Walk Through For A Single Company

We start with a ticker symbol and we use the *filing_list()* function from the [edgarWebR](https://github.com/mwaldstein/edgarWebR) library to retreive meta-information for this filing:

```r
ticker <- 'BJRI'
filing_list <- try(company_filings(as.character(ticker), ownership = FALSE, type = "10-K", before = "20180101",
                               count = 40, page = 1),TRUE)
isTRUE(class(filing_list)=='try-error')
[1] FALSE                               
```

We're off to a decent start because the function was able to find some information for the requested ticker symbol (I wrote this in a "try" statement because I discovered that are some valid ticker symbols for which meta-data could not be found).

So let's see what the *filing_list* object looks like:

```r
> filing_list
       accession_number  act file_number filing_date accepted_date
1  0001193125-17-059914   34   000-21423  2017-02-28    2017-02-27
2  0001193125-16-471871   34   000-21423  2016-02-23    2016-02-22
3  0001193125-15-065115   34   000-21423  2015-02-26    2015-02-26
4  0001193125-14-066983   34   000-21423  2014-02-25    2014-02-25
5  0001193125-13-074282   34   000-21423  2013-02-25    2013-02-25
6  0001193125-12-082223   34   000-21423  2012-02-28    2012-02-27
7  0001193125-11-061027   34   000-21423  2011-03-09    2011-03-09
8  0001193125-10-051099   34   000-21423  2010-03-09    2010-03-09
9  0001193125-09-054027   34   000-21423  2009-03-13    2009-03-13
10 0001193125-08-057707   34   000-21423  2008-03-17    2008-03-14
11 0001193125-07-053535   34   000-21423  2007-03-13    2007-03-13
12 0001193125-06-098178   34   000-21423  2006-05-03    2006-05-03
13 0001193125-06-056951   34   000-21423  2006-03-17    2006-03-16
14 0001193125-05-092385   34   000-21423  2005-05-02    2005-05-02
15 0001193125-05-050976   34   000-21423  2005-03-15    2005-03-15
16 0001193125-04-033937 <NA>   000-21423  2004-03-04    2004-03-03
17 0001104659-03-005174 <NA>   000-21423  2003-03-28    2003-03-27
18 0000912057-02-011513 <NA>   000-21423  2002-03-26    2002-03-26
19 0000912057-01-511305 <NA>   000-21423  2001-04-30    2001-04-30
20 0001013488-01-000004 <NA>   000-21423  2001-04-02    2001-04-02
21 0001013488-00-000004 <NA>   000-21423  2000-04-28    2000-04-28
22 0000912057-00-014882 <NA>   000-21423  2000-03-30    2000-03-30
                                                                                                href    type
1  https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/0001193125-17-059914-index.htm    10-K
2  https://www.sec.gov/Archives/edgar/data/1013488/000119312516471871/0001193125-16-471871-index.htm    10-K
3  https://www.sec.gov/Archives/edgar/data/1013488/000119312515065115/0001193125-15-065115-index.htm    10-K
4  https://www.sec.gov/Archives/edgar/data/1013488/000119312514066983/0001193125-14-066983-index.htm    10-K
5  https://www.sec.gov/Archives/edgar/data/1013488/000119312513074282/0001193125-13-074282-index.htm    10-K
6  https://www.sec.gov/Archives/edgar/data/1013488/000119312512082223/0001193125-12-082223-index.htm    10-K
7  https://www.sec.gov/Archives/edgar/data/1013488/000119312511061027/0001193125-11-061027-index.htm    10-K
8  https://www.sec.gov/Archives/edgar/data/1013488/000119312510051099/0001193125-10-051099-index.htm    10-K
9  https://www.sec.gov/Archives/edgar/data/1013488/000119312509054027/0001193125-09-054027-index.htm    10-K
10 https://www.sec.gov/Archives/edgar/data/1013488/000119312508057707/0001193125-08-057707-index.htm    10-K
11 https://www.sec.gov/Archives/edgar/data/1013488/000119312507053535/0001193125-07-053535-index.htm    10-K
12 https://www.sec.gov/Archives/edgar/data/1013488/000119312506098178/0001193125-06-098178-index.htm  10-K/A
13 https://www.sec.gov/Archives/edgar/data/1013488/000119312506056951/0001193125-06-056951-index.htm    10-K
14 https://www.sec.gov/Archives/edgar/data/1013488/000119312505092385/0001193125-05-092385-index.htm  10-K/A
15 https://www.sec.gov/Archives/edgar/data/1013488/000119312505050976/0001193125-05-050976-index.htm    10-K
16 https://www.sec.gov/Archives/edgar/data/1013488/000119312504033937/0001193125-04-033937-index.htm    10-K
17 https://www.sec.gov/Archives/edgar/data/1013488/000110465903005174/0001104659-03-005174-index.htm    10-K
18 https://www.sec.gov/Archives/edgar/data/1013488/000091205702011513/0000912057-02-011513-index.htm 10-K405
19 https://www.sec.gov/Archives/edgar/data/1013488/000091205701511305/0000912057-01-511305-index.htm  10-K/A
20 https://www.sec.gov/Archives/edgar/data/1013488/000101348801000004/0001013488-01-000004-index.htm    10-K
21                   https://www.sec.gov/Archives/edgar/data/1013488/0001013488-00-000004-index.html  10-K/A
22                   https://www.sec.gov/Archives/edgar/data/1013488/0000912057-00-014882-index.html 10-K405
   film_number                                              form_name description   size
1     17644520 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA>   5 MB
2    161446231 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA>   5 MB
3     15651600 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA>   7 MB
4     14639929 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA>   6 MB
5     13639816 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA>   6 MB
6     12643962 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA>   7 MB
7     11675949 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA>   1 MB
8     10666073 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA>   1 MB
9     09681355 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA>   1 MB
10    08690854 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA>   1 MB
11    07691133 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA> 827 KB
12    06803369 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA> 282 KB
13    06693497 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA> 769 KB
14    05791404 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA> 209 KB
15    05681631 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA> 930 KB
16    04647020 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA> 785 KB
17    03622383 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA>   4 MB
18    02585272    Annual report [Sections 13 and 15(d), S-K Item 405]        <NA>   1 MB
19     1615586 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA>  50 KB
20     1590138 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA> 706 KB
21      613381 Annual report [Section 13 and 15(d), not S-K Item 405]        <NA>  46 KB
22      586209    Annual report [Sections 13 and 15(d), S-K Item 405]        <NA> 420 KB
> 
```

The key result one should be aware of here is the column href:

```r
> filing_list$href
 [1] "https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/0001193125-17-059914-index.htm"
 [2] "https://www.sec.gov/Archives/edgar/data/1013488/000119312516471871/0001193125-16-471871-index.htm"
 [3] "https://www.sec.gov/Archives/edgar/data/1013488/000119312515065115/0001193125-15-065115-index.htm"
 [4] "https://www.sec.gov/Archives/edgar/data/1013488/000119312514066983/0001193125-14-066983-index.htm"
 [5] "https://www.sec.gov/Archives/edgar/data/1013488/000119312513074282/0001193125-13-074282-index.htm"
 [6] "https://www.sec.gov/Archives/edgar/data/1013488/000119312512082223/0001193125-12-082223-index.htm"
 [7] "https://www.sec.gov/Archives/edgar/data/1013488/000119312511061027/0001193125-11-061027-index.htm"
 [8] "https://www.sec.gov/Archives/edgar/data/1013488/000119312510051099/0001193125-10-051099-index.htm"
 [9] "https://www.sec.gov/Archives/edgar/data/1013488/000119312509054027/0001193125-09-054027-index.htm"
[10] "https://www.sec.gov/Archives/edgar/data/1013488/000119312508057707/0001193125-08-057707-index.htm"
[11] "https://www.sec.gov/Archives/edgar/data/1013488/000119312507053535/0001193125-07-053535-index.htm"
[12] "https://www.sec.gov/Archives/edgar/data/1013488/000119312506098178/0001193125-06-098178-index.htm"
[13] "https://www.sec.gov/Archives/edgar/data/1013488/000119312506056951/0001193125-06-056951-index.htm"
[14] "https://www.sec.gov/Archives/edgar/data/1013488/000119312505092385/0001193125-05-092385-index.htm"
[15] "https://www.sec.gov/Archives/edgar/data/1013488/000119312505050976/0001193125-05-050976-index.htm"
[16] "https://www.sec.gov/Archives/edgar/data/1013488/000119312504033937/0001193125-04-033937-index.htm"
[17] "https://www.sec.gov/Archives/edgar/data/1013488/000110465903005174/0001104659-03-005174-index.htm"
[18] "https://www.sec.gov/Archives/edgar/data/1013488/000091205702011513/0000912057-02-011513-index.htm"
[19] "https://www.sec.gov/Archives/edgar/data/1013488/000091205701511305/0000912057-01-511305-index.htm"
[20] "https://www.sec.gov/Archives/edgar/data/1013488/000101348801000004/0001013488-01-000004-index.htm"
[21] "https://www.sec.gov/Archives/edgar/data/1013488/0001013488-00-000004-index.html"                  
[22] "https://www.sec.gov/Archives/edgar/data/1013488/0000912057-00-014882-index.html"                  
> 
```
These are the index pages where the filing documents for each filing live.  

Next step is to get the filing details from these index pages:

```r

#parse this listing to find only 10-K filings...e.g. not 10-K/A and 10-K405
filing_list <- filing_list[filing_list$type=="10-K",]

filing.details <- filing_details(filing_list$href[1])

filing.details
$information
  type                                             description     accession_number filing_date
1 10-K Annual report [Section 13 and 15(d), not S-K Item 405]: 0001193125-17-059914  2017-02-28
        accepted_date documents period_date changed_date effective_date   bytes
1 2017-02-27 21:50:38        75  2017-01-03         <NA>           <NA> 5339245

$documents
   seq                                   description                  document
1    1                                     FORM 10-K           d280042d10k.htm
2    2                                      EX-10.23        d280042dex1023.htm
3    3                                         EX-21          d280042dex21.htm
4    4                                       EX-23.1         d280042dex231.htm
5    5                                         EX-31          d280042dex31.htm
6    6                                         EX-32          d280042dex32.htm
7   13                                       GRAPHIC g280042g0223145331454.jpg
8   14                                       GRAPHIC         g280042g46q82.jpg
9   NA                 Complete submission text file  0001193125-17-059914.txt
10   7                        XBRL INSTANCE DOCUMENT         bjri-20170103.xml
11   8                XBRL TAXONOMY EXTENSION SCHEMA         bjri-20170103.xsd
12   9  XBRL TAXONOMY EXTENSION CALCULATION LINKBASE     bjri-20170103_cal.xml
13  10   XBRL TAXONOMY EXTENSION DEFINITION LINKBASE     bjri-20170103_def.xml
14  11        XBRL TAXONOMY EXTENSION LABEL LINKBASE     bjri-20170103_lab.xml
15  12 XBRL TAXONOMY EXTENSION PRESENTATION LINKBASE     bjri-20170103_pre.xml
                                                                                           href       type
1            https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/d280042d10k.htm       10-K
2         https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/d280042dex1023.htm   EX-10.23
3           https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/d280042dex21.htm      EX-21
4          https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/d280042dex231.htm    EX-23.1
5           https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/d280042dex31.htm      EX-31
6           https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/d280042dex32.htm      EX-32
7  https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/g280042g0223145331454.jpg    GRAPHIC
8          https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/g280042g46q82.jpg    GRAPHIC
9   https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/0001193125-17-059914.txt           
10         https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/bjri-20170103.xml EX-101.INS
11         https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/bjri-20170103.xsd EX-101.SCH
12     https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/bjri-20170103_cal.xml EX-101.CAL
13     https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/bjri-20170103_def.xml EX-101.DEF
14     https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/bjri-20170103_lab.xml EX-101.LAB
15     https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/bjri-20170103_pre.xml EX-101.PRE
      size
1   991513
2    23003
3     2099
4     5496
5    15807
6     3781
7     3655
8    61712
9  5339245
10  970442
11   46917
12   87571
13  178318
14  466612
15  358246

$filers
   mailing_address_1 mailing_address_2         mailing_address_3 mailing_address_4 business_address_1
1 7755 CENTER AVENUE         SUITE 300 HUNTINGTON BEACH CA 92647              <NA> 7755 CENTER AVENUE
  business_address_2        business_address_3 business_address_4        company_name company_cik
1          SUITE 300 HUNTINGTON BEACH CA 92647     (714) 500-2440 BJs RESTAURANTS INC  0001013488
                                                       company_filings_href company_irs_number
1 https://www.sec.gov/cgi-bin/browse-edgar?CIK=0001013488&action=getcompany          330485615
  company_incorporation_state company_fiscal_year_end filing_type filing_act
1                          CA                    1230        10-K         34
                                                              file_number_href file_number film_number sic_code
1 https://www.sec.gov/cgi-bin/browse-edgar?filenum=000-21423&action=getcompany   000-21423    17644520     5812
                                                                           sic_href
1 https://www.sec.gov/cgi-bin/browse-edgar?action=getcompany&SIC=5812&owner=include

$funds
data frame with 0 columns and 0 rows

> 
```

The filing details give us another big list of urls which give us the locations of the various documents attached to each 10-K filing.  Again, the output that we are most interested in in the "href" column:

```r
> filing.details$documents$href
 [1] "https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/d280042d10k.htm"          
 [2] "https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/d280042dex1023.htm"       
 [3] "https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/d280042dex21.htm"         
 [4] "https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/d280042dex231.htm"        
 [5] "https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/d280042dex31.htm"         
 [6] "https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/d280042dex32.htm"         
 [7] "https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/g280042g0223145331454.jpg"
 [8] "https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/g280042g46q82.jpg"        
 [9] "https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/0001193125-17-059914.txt" 
[10] "https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/bjri-20170103.xml"        
[11] "https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/bjri-20170103.xsd"        
[12] "https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/bjri-20170103_cal.xml"    
[13] "https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/bjri-20170103_def.xml"    
[14] "https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/bjri-20170103_lab.xml"    
[15] "https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/bjri-20170103_pre.xml"    
> 
```

The document we ultimately want to feed into a parsing function is the XBRL INSTANCE DOCUMENT...most of the time this will be the 1st .xml file in the document list.  

Since, the file we want is generally the first .xml file, I use a regular expression search to find all the .xml files and then look for the first one in the list.

```r
docs <- filing.details$documents$href
xml.idx <- grep('.xml',docs)

  idx.min <- min(xml.idx)
  xml.file <- filing.details$documents$href[idx.min]
  
  > xml.file
[1] "https://www.sec.gov/Archives/edgar/data/1013488/000119312517059914/bjri-20170103.xml"


  xml.parsed <- try(xbrlDoAll(xml.file),TRUE)
result <-  xbrl_get_statements(xml.parsed)

class(result) 
[1] "statements" "list"

str(result)
List of 3
 $ StatementOfFinancialPositionClassified:Classes ‘statement’ and 'data.frame':	2 obs. of  29 variables:
  ..$ contextId                            : chr [1:2] "eol_PE2044----1710-K0003_STD_0_20151229_0" "eol_PE2044----1710-K0003_STD_0_20170103_0"
  ..$ startDate                            : chr [1:2] NA NA
  ..$ endDate                              : chr [1:2] "2015-12-29" "2017-01-03"
  ..$ decimals                             : num [1:2] -5 -5
  ..$ Assets                               : num [1:2] 6.82e+08 7.09e+08
  ..$ AssetsCurrent                        : num [1:2] 93003000 77073000
  ..$ CashAndCashEquivalentsAtCarryingValue: num [1:2] 34604000 22761000
  ..$ ReceivablesNetCurrent                : num [1:2] 25364000 14698000
  ..$ FIFOInventoryAmount                  : num [1:2] 8893000 9907000
  ..$ PrepaidExpenseAndOtherAssetsCurrent  : num [1:2] 7171000 11324000
  ..$ DeferredTaxAssetsNetCurrent          : num [1:2] 16971000 18383000
  ..$ PropertyPlantAndEquipmentNet         : num [1:2] 5.62e+08 6.01e+08
  ..$ Goodwill                             : num [1:2] 4673000 4673000
  ..$ OtherAssetsNoncurrent                : num [1:2] 22157000 25809000
  ..$ LiabilitiesAndStockholdersEquity     : num [1:2] 6.82e+08 7.09e+08
  ..$ Liabilities                          : num [1:2] 3.65e+08 4.34e+08
  ..$ LiabilitiesCurrent                   : num [1:2] 1.17e+08 1.26e+08
  ..$ AccountsPayableCurrent               : num [1:2] 33033000 31145000
  ..$ AccruedLiabilitiesCurrent            : num [1:2] 83861000 94553000
  ..$ DeferredTaxLiabilitiesNoncurrent     : num [1:2] 46669000 55154000
  ..$ DeferredRentCreditNoncurrent         : num [1:2] 27627000 30424000
  ..$ IncentiveFromLessor                  : num [1:2] 53837000 54119000
  ..$ LongTermDebt                         : num [1:2] 1.00e+08 1.48e+08
  ..$ OtherLiabilitiesNoncurrent           : num [1:2] 19655000 20587000
  ..$ StockholdersEquity                   : num [1:2] 3.16e+08 2.75e+08
  ..$ PreferredStockValue                  : num [1:2] 0 0
  ..$ CommonStockValue                     : num [1:2] 7367000 NA
  ..$ AdditionalPaidInCapitalCommonStock   : num [1:2] 63290000 66200000
  ..$ RetainedEarningsAccumulatedDeficit   : num [1:2] 2.46e+08 2.09e+08
  ..- attr(*, "role_id")= chr "StatementOfFinancialPositionClassified"
  ..- attr(*, "relations")=Classes ‘xbrl_relations’ and 'data.frame':	23 obs. of  3 variables:
  .. ..$ fromElementId: chr [1:23] "AssetsCurrent" "AssetsCurrent" "AssetsCurrent" "AssetsCurrent" ...
  .. ..$ toElementId  : chr [1:23] "CashAndCashEquivalentsAtCarryingValue" "ReceivablesNetCurrent" "FIFOInventoryAmount" "PrepaidExpenseAndOtherAssetsCurrent" ...
  .. ..$ order        : num [1:23] 1.01 1.02 1.03 1.04 1.05 1.06 1.07 1.08 1.09 1.1 ...
  ..- attr(*, "elements")=Classes ‘elements’ and 'data.frame':	25 obs. of  8 variables:
  .. ..$ elementId  : chr [1:25] "Assets" "AssetsCurrent" "CashAndCashEquivalentsAtCarryingValue" "ReceivablesNetCurrent" ...
  .. ..$ parentId   : chr [1:25] NA "Assets" "AssetsCurrent" "AssetsCurrent" ...
  .. ..$ order      : num [1:25] NA 1.06 1.01 1.02 1.03 1.04 1.05 1.07 1.08 1.09 ...
  .. ..$ balance    : chr [1:25] "debit" "debit" "debit" "debit" ...
  .. ..$ labelString: chr [1:25] "Assets" "Assets, Current" "Cash and Cash Equivalents, at Carrying Value" "Receivables, Net, Current" ...
  .. ..$ level      : num [1:25] 1 2 3 3 3 3 3 2 2 2 ...
  .. ..$ id         : chr [1:25] "01" "0101" "010101" "010102" ...
  .. ..$ terminal   : logi [1:25] FALSE FALSE TRUE TRUE TRUE TRUE ...
 $ StatementOfIncomeAlternative          :Classes ‘statement’ and 'data.frame':	2 obs. of  22 variables:
  ..$ contextId                                                                                  : chr [1:2] "eol_PE2044----1710-K0003_STD_364_20141230_0" "eol_PE2044----1710-K0003_STD_371_20170103_0"
  ..$ startDate                                                                                  : chr [1:2] "2014-01-01" "2015-12-30"
  ..$ endDate                                                                                    : chr [1:2] "2014-12-30" "2017-01-03"
  ..$ decimals                                                                                   : num [1:2] -5 -5
  ..$ NetIncomeLoss                                                                              : num [1:2] 27397000 45557000
  ..$ IncomeLossFromContinuingOperationsBeforeIncomeTaxesExtraordinaryItemsNoncontrollingInterest: num [1:2] 36323000 61091000
  ..$ OperatingIncomeLoss                                                                        : num [1:2] 35426000 61641000
  ..$ SalesRevenueGoodsNet                                                                       : num [1:2] 8.46e+08 9.93e+08
  ..$ CostsAndExpenses                                                                           : num [1:2] 8.10e+08 9.31e+08
  ..$ CostOfGoodsSold                                                                            : num [1:2] 2.13e+08 2.51e+08
  ..$ LaborAndRelatedExpense                                                                     : num [1:2] 2.99e+08 3.45e+08
  ..$ bjri_OccupancyAndOperatingCosts                                                            : num [1:2] 1.82e+08 2.05e+08
  ..$ GeneralAndAdministrativeExpense                                                            : num [1:2] 51558000 55373000
  ..$ DepreciationDepletionAndAmortization                                                       : num [1:2] 55387000 64275000
  ..$ PreOpeningCosts                                                                            : num [1:2] 4973000 6977000
  ..$ GainLossOnSalesOfAssetsAndAssetImpairmentCharges                                           : num [1:2] -1963000 -2971000
  ..$ GainLossOnContractTermination                                                              : num [1:2] NA NA
  ..$ bjri_LegalAndOtherExpenses                                                                 : num [1:2] 2431000 402000
  ..$ NonoperatingIncomeExpense                                                                  : num [1:2] 897000 -550000
  ..$ InterestIncomeExpenseNet                                                                   : num [1:2] -238000 -1730000
  ..$ OtherNonoperatingIncomeExpense                                                             : num [1:2] 1135000 1180000
  ..$ IncomeTaxExpenseBenefit                                                                    : num [1:2] 8926000 15534000
  ..- attr(*, "role_id")= chr "StatementOfIncomeAlternative"
  ..- attr(*, "relations")=Classes ‘xbrl_relations’ and 'data.frame':	17 obs. of  3 variables:
  .. ..$ fromElementId: chr [1:17] "OperatingIncomeLoss" "CostsAndExpenses" "CostsAndExpenses" "CostsAndExpenses" ...
  .. ..$ toElementId  : chr [1:17] "SalesRevenueGoodsNet" "CostOfGoodsSold" "LaborAndRelatedExpense" "bjri_OccupancyAndOperatingCosts" ...
  .. ..$ order        : num [1:17] 1.01 1.02 1.03 1.04 1.05 1.06 1.07 1.08 1.09 1.1 ...
  ..- attr(*, "elements")=Classes ‘elements’ and 'data.frame':	18 obs. of  8 variables:
  .. ..$ elementId  : chr [1:18] "NetIncomeLoss" "IncomeLossFromContinuingOperationsBeforeIncomeTaxesExtraordinaryItemsNoncontrollingInterest" "OperatingIncomeLoss" "SalesRevenueGoodsNet" ...
  .. ..$ parentId   : chr [1:18] NA "NetIncomeLoss" "IncomeLossFromContinuingOperationsBeforeIncomeTaxesExtraordinaryItemsNoncontrollingInterest" "OperatingIncomeLoss" ...
  .. ..$ order      : num [1:18] NA 1.16 1.12 1.01 1.11 1.02 1.03 1.04 1.05 1.06 ...
  .. ..$ balance    : chr [1:18] "credit" "credit" "credit" "credit" ...
  .. ..$ labelString: chr [1:18] "Net Income (Loss) Attributable to Parent" "Income (Loss) from Continuing Operations before Income Taxes, Noncontrolling Interest" "Operating Income (Loss)" "Sales Revenue, Goods, Net" ...
  .. ..$ level      : num [1:18] 1 2 3 4 4 5 5 5 5 5 ...
  .. ..$ id         : chr [1:18] "01" "0101" "010101" "01010101" ...
  .. ..$ terminal   : logi [1:18] FALSE FALSE FALSE TRUE FALSE TRUE ...
 $ StatementOfCashFlowsIndirect          :Classes ‘statement’ and 'data.frame':	3 obs. of  34 variables:
  ..$ contextId                                                     : chr [1:3] "eol_PE2044----1710-K0003_STD_364_20141230_0" "eol_PE2044----1710-K0003_STD_364_20151229_0" "eol_PE2044----1710-K0003_STD_371_20170103_0"
  ..$ startDate                                                     : chr [1:3] "2014-01-01" "2014-12-31" "2015-12-30"
  ..$ endDate                                                       : chr [1:3] "2014-12-30" "2015-12-29" "2017-01-03"
  ..$ decimals                                                      : num [1:3] -5 -5 -5
  ..$ CashAndCashEquivalentsPeriodIncreaseDecrease                  : num [1:3] 7688000 3921000 -11843000
  ..$ NetCashProvidedByUsedInOperatingActivitiesContinuingOperations: num [1:3] 1.00e+08 1.27e+08 1.38e+08
  ..$ NetIncomeLoss                                                 : num [1:3] 27397000 45325000 45557000
  ..$ DepreciationDepletionAndAmortization                          : num [1:3] 55387000 59417000 64275000
  ..$ DeferredIncomeTaxesAndTaxCredits                              : num [1:3] 4416000 5319000 7073000
  ..$ ShareBasedCompensation                                        : num [1:3] 4855000 5395000 5527000
  ..$ GainLossOnSalesOfAssetsAndAssetImpairmentCharges              : num [1:3] -1963000 -2908000 -2971000
  ..$ GainLossOnContractTermination                                 : num [1:3] NA 2910000 NA
  ..$ IncreaseDecreaseInAccountsAndOtherReceivables                 : num [1:3] 5393000 994000 -9904000
  ..$ IncreaseDecreaseInAccountsReceivableAndOtherOperatingAssets   : num [1:3] 627000 -426000 -762000
  ..$ IncreaseDecreaseInInventories                                 : num [1:3] 577000 883000 1014000
  ..$ IncreaseDecreaseInPrepaidDeferredExpenseAndOtherAssets        : num [1:3] 1662000 -1477000 5065000
  ..$ IncreaseDecreaseInOtherOperatingAssets                        : num [1:3] 2706000 3282000 5257000
  ..$ IncreaseDecreaseInAccountsPayable                             : num [1:3] 842000 -1983000 542000
  ..$ IncreaseDecreaseInAccruedLiabilities                          : num [1:3] 12179000 11274000 10692000
  ..$ IncreaseDecreaseInDeferredCharges                             : num [1:3] -2532000 -2947000 -2797000
  ..$ IncreaseDecreaseInDeferredRevenue                             : num [1:3] -248000 2753000 282000
  ..$ IncreaseDecreaseInOtherOperatingLiabilities                   : num [1:3] 1682000 35000 -687000
  ..$ NetCashProvidedByUsedInInvestingActivitiesContinuingOperations: num [1:3] -6.52e+07 -8.26e+07 -1.05e+08
  ..$ PaymentsToAcquirePropertyPlantAndEquipment                    : num [1:3] 8.81e+07 8.61e+07 1.09e+08
  ..$ ProceedsFromSaleOfProductiveAssets                            : num [1:3] 13143000 3478000 4511000
  ..$ ProceedsFromSaleAndMaturityOfMarketableSecurities             : num [1:3] 1.9e+07 NA NA
  ..$ PaymentsToAcquireMarketableSecurities                         : num [1:3] 9159000 NA NA
  ..$ NetCashProvidedByUsedInFinancingActivitiesContinuingOperations: num [1:3] -27162000 -40711000 -45350000
  ..$ ProceedsFromLinesOfCredit                                     : num [1:3] 1.25e+08 5.29e+08 1.18e+09
  ..$ RepaymentsOfLinesOfCredit                                     : num [1:3] 6.70e+07 4.87e+08 1.13e+09
  ..$ ExcessTaxBenefitFromShareBasedCompensationFinancingActivities : num [1:3] 3803000 4220000 333000
  ..$ PaymentsRelatedToTaxWithholdingForShareBasedCompensation      : num [1:3] 445000 293000 323000
  ..$ ProceedsFromStockOptionsExercised                             : num [1:3] 11480000 8411000 2126000
  ..$ PaymentsForRepurchaseOfCommonStock                            : num [1:3] 1.00e+08 9.55e+07 9.50e+07
  ..- attr(*, "role_id")= chr "StatementOfCashFlowsIndirect"
  ..- attr(*, "relations")=Classes ‘xbrl_relations’ and 'data.frame':	29 obs. of  3 variables:
  .. ..$ fromElementId: chr [1:29] "NetCashProvidedByUsedInOperatingActivitiesContinuingOperations" "NetCashProvidedByUsedInOperatingActivitiesContinuingOperations" "NetCashProvidedByUsedInOperatingActivitiesContinuingOperations" "NetCashProvidedByUsedInOperatingActivitiesContinuingOperations" ...
  .. ..$ toElementId  : chr [1:29] "NetIncomeLoss" "DepreciationDepletionAndAmortization" "DeferredIncomeTaxesAndTaxCredits" "ShareBasedCompensation" ...
  .. ..$ order        : num [1:29] 1.01 1.02 1.03 1.04 1.05 1.06 1.07 1.08 1.09 1.1 ...
  ..- attr(*, "elements")=Classes ‘elements’ and 'data.frame':	30 obs. of  8 variables:
  .. ..$ elementId  : chr [1:30] "CashAndCashEquivalentsPeriodIncreaseDecrease" "NetCashProvidedByUsedInOperatingActivitiesContinuingOperations" "NetIncomeLoss" "DepreciationDepletionAndAmortization" ...
  .. ..$ parentId   : chr [1:30] NA "CashAndCashEquivalentsPeriodIncreaseDecrease" "NetCashProvidedByUsedInOperatingActivitiesContinuingOperations" "NetCashProvidedByUsedInOperatingActivitiesContinuingOperations" ...
  .. ..$ order      : num [1:30] NA 1.17 1.01 1.02 1.03 1.04 1.05 1.06 1.07 1.08 ...
  .. ..$ balance    : chr [1:30] "debit" NA "credit" "debit" ...
  .. ..$ labelString: chr [1:30] "Cash and Cash Equivalents, Period Increase (Decrease)" "Net Cash Provided by (Used in) Operating Activities, Continuing Operations" "Net Income (Loss) Attributable to Parent" "Depreciation, Depletion and Amortization" ...
  .. ..$ level      : num [1:30] 1 2 3 3 3 3 3 3 3 3 ...
  .. ..$ id         : chr [1:30] "01" "0101" "010101" "010102" ...
  .. ..$ terminal   : logi [1:30] FALSE FALSE TRUE TRUE TRUE TRUE ...
 - attr(*, "class")= chr [1:2] "statements" "list"
```

As you can see, the list object returned by xbrl_get_statement() has a lot of info.  We can simplify life a little bit by looking at one list element at at time

```r
result

Financial statements repository
                                             From         To Rows Columns
StatementOfFinancialPositionClassified 2015-12-29 2017-01-03    2      29
StatementOfIncomeAlternative           2014-12-30 2017-01-03    2      22
StatementOfCashFlowsIndirect           2014-12-30 2017-01-03    3      34

result$StatementofIncomeAlternative
> result$StatementOfIncomeAlternative
Financial statement: 2 observations from 2014-12-30 to 2017-01-03 
 Element                                                  2017-01-03 2014-12-30
 NetIncomeLoss =                                           455.57     273.97   
 + IncomeLossFromContinuingOperationsBefore... =           610.91     363.23   
   + OperatingIncomeLoss =                                 616.41     354.26   
     + SalesRevenueGoodsNet                               9930.52    8455.69   
     - CostsAndExpenses =                                 9314.11    8101.43   
       + CostOfGoodsSold                                  2514.60    2129.79   
       + LaborAndRelatedExpense                           3453.70    2987.03   
       + bjri_OccupancyAndOperatingCosts                  2045.83    1821.49   
       + GeneralAndAdministrativeExpense                   553.73     515.58   
       + DepreciationDepletionAndAmortization              642.75     553.87   
       + PreOpeningCosts                                    69.77      49.73   
       - GainLossOnSalesOfAssetsAndAssetImpairmentCharges  -29.71     -19.63   
       - GainLossOnContractTermination                         NA         NA   
       + bjri_LegalAndOtherExpenses                          4.02      24.31   
   + NonoperatingIncomeExpense =                            -5.50       8.97   
     + InterestIncomeExpenseNet                            -17.30      -2.38   
     + OtherNonoperatingIncomeExpense                       11.80      11.35   
 - IncomeTaxExpenseBenefit                                 155.34      89.26   
 
> str(result$StatementOfIncomeAlternative)

Classes ‘statement’ and 'data.frame':	2 obs. of  22 variables:
 $ contextId                                                                                  : chr  "eol_PE2044----1710-K0003_STD_364_20141230_0" "eol_PE2044----1710-K0003_STD_371_20170103_0"
 $ startDate                                                                                  : chr  "2014-01-01" "2015-12-30"
 $ endDate                                                                                    : chr  "2014-12-30" "2017-01-03"
 $ decimals                                                                                   : num  -5 -5
 $ NetIncomeLoss                                                                              : num  27397000 45557000
 $ IncomeLossFromContinuingOperationsBeforeIncomeTaxesExtraordinaryItemsNoncontrollingInterest: num  36323000 61091000
 $ OperatingIncomeLoss                                                                        : num  35426000 61641000
 $ SalesRevenueGoodsNet                                                                       : num  8.46e+08 9.93e+08
 $ CostsAndExpenses                                                                           : num  8.10e+08 9.31e+08
 $ CostOfGoodsSold                                                                            : num  2.13e+08 2.51e+08
 $ LaborAndRelatedExpense                                                                     : num  2.99e+08 3.45e+08
 $ bjri_OccupancyAndOperatingCosts                                                            : num  1.82e+08 2.05e+08
 $ GeneralAndAdministrativeExpense                                                            : num  51558000 55373000
 $ DepreciationDepletionAndAmortization                                                       : num  55387000 64275000
 $ PreOpeningCosts                                                                            : num  4973000 6977000
 $ GainLossOnSalesOfAssetsAndAssetImpairmentCharges                                           : num  -1963000 -2971000
 $ GainLossOnContractTermination                                                              : num  NA NA
 $ bjri_LegalAndOtherExpenses                                                                 : num  2431000 402000
 $ NonoperatingIncomeExpense                                                                  : num  897000 -550000
 $ InterestIncomeExpenseNet                                                                   : num  -238000 -1730000
 $ OtherNonoperatingIncomeExpense                                                             : num  1135000 1180000
 $ IncomeTaxExpenseBenefit                                                                    : num  8926000 15534000
 - attr(*, "role_id")= chr "StatementOfIncomeAlternative"
 - attr(*, "relations")=Classes ‘xbrl_relations’ and 'data.frame':	17 obs. of  3 variables:
  ..$ fromElementId: chr  "OperatingIncomeLoss" "CostsAndExpenses" "CostsAndExpenses" "CostsAndExpenses" ...
  ..$ toElementId  : chr  "SalesRevenueGoodsNet" "CostOfGoodsSold" "LaborAndRelatedExpense" "bjri_OccupancyAndOperatingCosts" ...
  ..$ order        : num  1.01 1.02 1.03 1.04 1.05 1.06 1.07 1.08 1.09 1.1 ...
 - attr(*, "elements")=Classes ‘elements’ and 'data.frame':	18 obs. of  8 variables:
  ..$ elementId  : chr  "NetIncomeLoss" "IncomeLossFromContinuingOperationsBeforeIncomeTaxesExtraordinaryItemsNoncontrollingInterest" "OperatingIncomeLoss" "SalesRevenueGoodsNet" ...
  ..$ parentId   : chr  NA "NetIncomeLoss" "IncomeLossFromContinuingOperationsBeforeIncomeTaxesExtraordinaryItemsNoncontrollingInterest" "OperatingIncomeLoss" ...
  ..$ order      : num  NA 1.16 1.12 1.01 1.11 1.02 1.03 1.04 1.05 1.06 ...
  ..$ balance    : chr  "credit" "credit" "credit" "credit" ...
  ..$ labelString: chr  "Net Income (Loss) Attributable to Parent" "Income (Loss) from Continuing Operations before Income Taxes, Noncontrolling Interest" "Operating Income (Loss)" "Sales Revenue, Goods, Net" ...
  ..$ level      : num  1 2 3 4 4 5 5 5 5 5 ...
  ..$ id         : chr  "01" "0101" "010101" "01010101" ...
  ..$ terminal   : logi  FALSE FALSE FALSE TRUE FALSE TRUE ...
```

# Try to Generalize 

So I think I've demonstrated that this little pipeline can get some useful data for one company at a time.  Now the question (just as with our last post) is can I get ALL the data for ALL the years I want and ALL the companies I want?  The short answer is NO...but I can get kind of close.  I haven't worked out all the kinks of getting all 62 companies in a single swing with this new approach yet, but it does appear that the new approaches gets me a bit more information than the old one.

Before dropping the whole routine on you let me point out the 2 main places where the crawler can break down:


1.  company_filings() - breakdowns here were pretty rare for me...but there were a few tickers that didn't exist.  
2. .xml search - some 10-K filings did not have any .xml files in their document libraries.  In this case the routine breaks down because it is specifically looking for the XBRL INSTANCE DOCUMENT to parse.  If this doc doesn't exist, we're hosed.
3. xbrlDoAll() - the most common reason for errors in this function call appear to me to be time out errors where - for whatever reason - the url request just couldn't be put through.

In each of the three cases above I used some really basic error handling that you can see below.

## A function to get one company at a time

```r
get.fins <- function(ticker){
#---------------------------------------------------------------------------

# get the list of available filings for the company of interest
filing_list <- try(company_filings(as.character(ticker), ownership = FALSE, type = "10-K", before = "20180101",
                               count = 40, page = 1),TRUE)

if(isTRUE(class(filing_list)=='try-error')){
  return(list(list(ticker),list("can't find company"),list("can't find company")))
}else{

#parse this listing to find only 10-K filings...e.g. not 10-K/A and 10-K405
filing_list <- filing_list[filing_list$type=="10-K",]
#-----------------------------------------------------------------------------

#----------------------------------------------------------------------------
# loop through the index files to get info for each filing
fin.statements <- list()
for(i in 1:length(filing_list$href)){

# the column 'href' in filing list has the file path to the index files we want
filing.details <- filing_details(filing_list$href[i])

#find the index of the first .xml file in the href list
docs <- filing.details$documents$href
xml.idx <- grep('.xml',docs)

result <- NULL
if(length(xml.idx)==0){
  result <- data.frame(error='no xml file',index=filing_list$href[i])
}else{
  idx.min <- min(xml.idx)
  xml.file <- filing.details$documents$href[idx.min]
  xml.parsed <- try(xbrlDoAll(xml.file),TRUE)
    if(isTRUE(class(xml.parsed)=='try-error')){
      result <- data.frame(error='time-out',index=filing_list$href[i])
    }else{
      result <- xbrl_get_statements(xml.parsed)
    }
}

fin.statements[[i]] <- result
}

errors.df <- list()
stats.df <- list()
k.err <- 0
k.stat <- 0
for(ilist in 1:length(fin.statements)){

  if(class(fin.statements[[ilist]])[1]=='data.frame'){
    k.err <- k.err + 1
    errors.df[[k.err]]<- fin.statements[[ilist]]
  }
  
  if(class(fin.statements[[ilist]])[1]=='statements'){
    k.stat <- k.stat + 1
    stats.df[[k.stat]] <- fin.statements[[ilist]]
  }

}

return(list(ticker,stats.df,errors.df))
}

}
```

## A comparison

Here's a couple comparisons between what we got for a few of the pricklier companies on my list using the edgarWebR/XBRL/finstr approach versus the approach from [last post](https://aaronmams.github.io/Accessing-Financial-Data-from-the-SEC-Part-1/) which relies on the finreportr package:

```r
library(finreportr)

ticker <- 'BLMN'
blmn <- get.fins('BLMN')

> test
[[1]]
[1] "BLMN"

[[2]]
[[2]][[1]]
Financial statements repository
                                                               From         To Rows Columns
ConsolidatedBalanceSheets                                2015-12-27 2016-12-25    2      38
ConsolidatedStatementsOfCashFlows                        2014-12-28 2016-12-25    3      50
ConsolidatedStatementsOfOperationsAndComprehensiveIncome 2014-12-28 2016-12-25    3      29

[[2]][[2]]
Financial statements repository
                                                               From         To Rows Columns
ConsolidatedBalanceSheets                                2014-12-28 2015-12-27    2      39
ConsolidatedStatementsOfCashFlows                        2013-12-31 2015-12-27    3      57
ConsolidatedStatementsOfOperationsAndComprehensiveIncome 2013-12-31 2015-12-27    3      32

[[2]][[3]]
Financial statements repository
                                                               From         To Rows Columns
ConsolidatedBalanceSheets                                2013-12-31 2014-12-28    2      41
ConsolidatedStatementsOfCashFlows                        2012-12-31 2014-12-28    3      60
ConsolidatedStatementsOfOperationsAndComprehensiveIncome 2012-12-31 2014-12-28    3      31

[[2]][[4]]
Financial statements repository
                                                               From         To Rows Columns
ConsolidatedBalanceSheets                                2012-12-31 2013-12-31    2      40
ConsolidatedStatementsOfCashFlows                        2011-12-31 2013-12-31    3      62
ConsolidatedStatementsOfOperationsAndComprehensiveIncome 2011-12-31 2013-12-31    3      31

[[2]][[5]]
Financial statements repository
                                                               From         To Rows Columns
ConsolidatedBalanceSheets                                2011-12-31 2012-12-31    2      40
ConsolidatedStatementsOfCashFlows                        2010-12-31 2012-12-31    3      61
ConsolidatedStatementsOfOperationsAndComprehensiveIncome 2010-12-31 2012-12-31    3      29


[[3]]
list()

> blmn.inc <- blmn[[2]]
> blmn.inc
[[1]]
Financial statements repository
                                                               From         To Rows Columns
ConsolidatedBalanceSheets                                2015-12-27 2016-12-25    2      38
ConsolidatedStatementsOfCashFlows                        2014-12-28 2016-12-25    3      50
ConsolidatedStatementsOfOperationsAndComprehensiveIncome 2014-12-28 2016-12-25    3      29

[[2]]
Financial statements repository
                                                               From         To Rows Columns
ConsolidatedBalanceSheets                                2014-12-28 2015-12-27    2      39
ConsolidatedStatementsOfCashFlows                        2013-12-31 2015-12-27    3      57
ConsolidatedStatementsOfOperationsAndComprehensiveIncome 2013-12-31 2015-12-27    3      32

[[3]]
Financial statements repository
                                                               From         To Rows Columns
ConsolidatedBalanceSheets                                2013-12-31 2014-12-28    2      41
ConsolidatedStatementsOfCashFlows                        2012-12-31 2014-12-28    3      60
ConsolidatedStatementsOfOperationsAndComprehensiveIncome 2012-12-31 2014-12-28    3      31

[[4]]
Financial statements repository
                                                               From         To Rows Columns
ConsolidatedBalanceSheets                                2012-12-31 2013-12-31    2      40
ConsolidatedStatementsOfCashFlows                        2011-12-31 2013-12-31    3      62
ConsolidatedStatementsOfOperationsAndComprehensiveIncome 2011-12-31 2013-12-31    3      31

[[5]]
Financial statements repository
                                                               From         To Rows Columns
ConsolidatedBalanceSheets                                2011-12-31 2012-12-31    2      40
ConsolidatedStatementsOfCashFlows                        2010-12-31 2012-12-31    3      61
ConsolidatedStatementsOfOperationsAndComprehensiveIncome 2010-12-31 2012-12-31    3      29

> bmn.inc[[1]]$ConsolidatedStatementsOfOperationsAndComprehensiveIncome
Financial statement: 3 observations from 2014-12-28 to 2016-12-25 
 Element                                                          2016-12-25 2015-12-27 2014-12-28
 NetIncomeLoss =                                                    417.48    1273.27     910.90  
 - NetIncomeLossAttributableToNoncontrollingInterest                 45.99      42.33      48.36  
 ComprehensiveIncomeNetOfTax =                                      779.71     405.02     569.66  
 + ComprehensiveIncomeNetOfTaxIncludingPort... =                    859.79     315.68     618.02  
   + ProfitLoss =                                                   463.47    1315.60     959.26  
     + IncomeLossFromContinuingOperationsBefore... =                564.91    1708.54    1199.70  
       + OperatingIncomeLoss =                                     1276.06    2309.25    1919.64  
         + Revenues =                                             42523.12   43776.76   44427.11  
           + SalesRevenueNet                                      42260.57   43499.21   44157.83  
           + OtherSalesRevenueNet                                   262.55     277.55     269.28  
         - CostsAndExpenses =                                     41247.06   41467.51   42507.47  
           + CostOfGoodsAndServicesSold                           13548.53   14196.89   14353.59  
           + CostOfGoodsSoldDirectLabor                           12112.50   12056.10   12189.61  
           + OtherCostAndExpenseOperating                          9921.57   10067.72   10490.53  
           + DepreciationDepletionAndAmortization                  1938.38    1903.99    1909.11  
           + GeneralAndAdministrativeExpense                       2679.81    2876.14    3043.82  
           + blmn_ProvisionforImpairedAssetsandRestaurantClosings  1046.27     366.67     520.81  
       + blmn_GainLossonExtinguishmentandModificationofDebt        -269.98     -29.56    -110.92  
       + OtherNonoperatingIncomeExpense                              16.09      -9.39     -12.44  
       + InterestIncomeExpenseNonoperatingNet                      -457.26    -561.76    -596.58  
     - IncomeTaxExpenseBenefit                                      101.44     392.94     240.44  
   + OtherComprehensiveIncomeLossForeignCurre...                    370.75    -961.94    -317.31  
   - OtherComprehensiveIncomeLossReclassifica...                    -38.07     -22.35       0.00  
   + OtherComprehensiveIncomeUnrealizedGainLo...                    -12.50     -60.33     -23.93  
 - ComprehensiveIncomeNetOfTaxAttributableT...                       80.08     -89.34      48.36  
>

# compare this to the finreportr output for 'BLMN'

> GetIncome('BLMN',2015)
Error in filter_impl(.data, quo) : Result must have length 1460, not 0
> GetIncome('BLMN',2016)
Error in filter_impl(.data, quo) : Result must have length 1925, not 0
> GetIncome('BLMN',2014)
Error in filter_impl(.data, quo) : Result must have length 1590, not 0
> 

```

So the edgarWebR/XBRL/finstr pipeline gave me statements for [Bloomin' Brands](https://www.sec.gov/cgi-bin/browse-edgar?CIK=blmn&owner=exclude&action=getcompany) from 2012-2016.  The finreportr approach gave me less than jack shit....so that's kinda cool.

Let's try another one. I'll try Dunkin Brands because the previous approach got almost all of thier data but was missing a rando year (2012).  Let's see what the new pipeline gives us:

```r
> dnkn.inc<- dnkn[[2]]
> dnkn.inc
[[1]]
Financial statements repository
                                                  From         To Rows Columns
ConsolidatedBalanceSheets                   2015-12-26 2016-12-31    2      43
ConsolidatedStatementsOfCashFlows           2014-12-27 2016-12-31    3      41
ConsolidatedStatementsOfComprehensiveIncome 2014-12-27 2016-12-31    3      13
ConsolidatedStatementsOfOperations          2014-12-27 2016-12-31    3      33

[[2]]
Financial statements repository
                                                  From         To Rows Columns
ConsolidatedBalanceSheets                   2014-12-27 2015-12-26    2      44
ConsolidatedStatementsOfCashFlows           2013-12-28 2015-12-26    3      43
ConsolidatedStatementsOfComprehensiveIncome 2013-12-28 2015-12-26    3      13
ConsolidatedStatementsOfOperations          2013-12-28 2015-12-26    3      32

[[3]]
Financial statements repository
                                                  From         To Rows Columns
ConsolidatedBalanceSheets                   2013-12-28 2014-12-27    2      42
ConsolidatedStatementsOfCashFlows           2012-12-29 2014-12-27    3      41
ConsolidatedStatementsOfComprehensiveIncome 2012-12-29 2014-12-27    3      13
ConsolidatedStatementsOfOperations          2012-12-29 2014-12-27    3      33

[[4]]
Financial statements repository
                                                  From         To Rows Columns
ConsolidatedBalanceSheets                   2012-12-29 2013-12-28    2      46
ConsolidatedStatementsOfCashFlows           2011-12-31 2013-12-28    3      42
ConsolidatedStatementsOfComprehensiveIncome 2011-12-31 2013-12-28    3      13
ConsolidatedStatementsOfOperations          2011-12-31 2013-12-28    3      33

[[5]]
Financial statements repository
                                                  From         To Rows Columns
ConsolidatedBalanceSheets                   2011-12-31 2012-12-29    2      45
ConsolidatedStatementsOfCashFlows           2010-12-25 2012-12-29    3      42
ConsolidatedStatementsOfComprehensiveIncome 2010-12-25 2012-12-29    3      13
ConsolidatedStatementsOfOperations          2010-12-25 2012-12-29    3      32

[[6]]
Financial statements repository
                                                           From         To Rows Columns
StatementConsolidatedBalanceSheets                   2010-12-25 2011-12-31    2      44
StatementConsolidatedStatementsOfOperations          2011-12-31 2011-12-31    1      28
StatementConsolidatedStatementsOfComprehensiveIncome 2009-12-26 2011-12-31    3       9
StatementConsolidatedStatementsOfCashFlows           2009-12-26 2011-12-31    3      43


```
So I definitely got info for the years ending 2011, 2012, and 2013.  Let's double check to see if our tempermental finreportr packages gives us this same info if we call it again:

```r
dnkn2 <- GetIncome('DNKN',2013)
dnkn2 <- tbl_df(dnkn2) %>% mutate(endDate=as.Date(endDate,format="%Y-%m-%d"),endyear=year(endDate)) %>%
            filter(endyear==2012)
            # A tibble: 54 x 6
   Metric                                            Units Amount    startDate  endDate    endyear
   <chr>                                             <chr> <chr>     <chr>      <date>       <int>
 1 Franchise Revenue                                 usd   418940000 2012-01-01 2012-12-29    2012
 2 Operating Leases, Income Statement, Lease Revenue usd   96816000  2012-01-01 2012-12-29    2012
 3 Sales Revenue, Goods, Net                         usd   94659000  2012-01-01 2012-12-29    2012
 4 Revenue from Franchisor Owned Outlets             usd   22922000  2012-01-01 2012-12-29    2012
 5 Other Revenue, Net                                usd   24844000  2012-01-01 2012-12-29    2012
 6 Revenues                                          usd   152372000 2012-01-01 2012-03-31    2012
 7 Revenues                                          usd   172387000 2012-04-01 2012-06-30    2012
 8 Revenues                                          usd   171719000 2012-07-01 2012-09-29    2012
 9 Revenues                                          usd   161703000 2012-09-30 2012-12-29    2012
10 Revenues                                          usd   658181000 2012-01-01 2012-12-29    2012
# ... with 44 more rows
> 
```

Weird...when I tried grabbing Dunkin' Brands before, *finreportr::GetIncome()* gave me everything except 2012.  Maybe this has to do with how I called the functions - recall that I used *finreportr::GetIncome()* for 2012, 2016, and 2018. And since the function is supposed to return everything BEFORE the specified data I figured I would get everything available from roughly 2008 to now.  Maybe *finreportr::GetIncome()* is more tempermental with respect to the date input that I realize. Or maybe the 2012 request just timed-out when I ran the batch before.

Ok, I'll do one more than wrap this thing up:  [Texas Roadhouse: TXRH](https://www.sec.gov/cgi-bin/browse-edgar?action=getcompany&CIK=0001289460&type=10-K&dateb=20180101&owner=exclude&count=40).

```r
#use the handrolled functions to get filing data from TXRH
txrh <- get.fins('TXRH')

> txrh[[2]]
[[1]]
Financial statements repository
                                                                    From         To Rows Columns
StatementConsolidatedBalanceSheets                            2015-12-29 2016-12-27    2      40
StatementConsolidatedStatementsOfIncomeAndComprehensiveIncome 2014-12-30 2016-12-27    3      28
StatementConsolidatedStatementsOfCashFlows                    2014-12-30 2016-12-27    3      44

[[2]]
Financial statements repository
                                                                 From         To Rows Columns
StatementCondensedConsolidatedBalanceSheets                2014-12-30 2015-12-29    2      40
StatementCondensedStatementsOfIncomeAndComprehensiveIncome 2013-12-31 2015-12-29    3      29
StatementCondensedStatementsOfCashFlows                    2013-12-31 2015-12-29    3      47

[[3]]
Financial statements repository
                                                                    From         To Rows Columns
StatementConsolidatedBalanceSheets                            2013-12-31 2014-12-30    2      40
StatementConsolidatedStatementsOfIncomeAndComprehensiveIncome 2012-12-25 2014-12-30    3      29
StatementConsolidatedStatementsOfCashFlows                    2012-12-25 2014-12-30    3      47

[[4]]
Financial statements repository
                                    From         To Rows Columns
BalanceSheet                  2012-12-25 2013-12-31    2      40
StatementOfIncome             2013-12-31 2013-12-31    1      27
StatementOfStockholdersEquity       <NA>       <NA>    0      22
CashFlows                     2013-12-31 2013-12-31    1      46

[[5]]
Financial statements repository
                                    From         To Rows Columns
BalanceSheet                  2011-12-27 2012-12-25    2      40
StatementOfIncome             2010-12-28 2012-12-25    3      26
StatementOfStockholdersEquity 2011-12-27 2012-12-25    2      22
CashFlows                     2010-12-28 2012-12-25    3      44

[[6]]
Financial statements repository
                                                               From         To Rows Columns
StatementOfIncome                                        2009-12-29 2011-12-27    3      24
StatementOfStockholdersEquityAndComprehensiveIncome      2011-12-27 2011-12-27    1      22
CashFlows                                                2009-12-29 2011-12-27    3      43
StatementOfStockholdersEquityAndComprehensiveIncomeCalc2 2009-12-29 2011-12-27    3       7
BalanceSheets                                            2010-12-28 2011-12-27    2      40

```

We got 2009 - 2015 so that's a decent start.  What do we get with *finreportr::GetIncome()*?

```r
> txrh2 <- GetIncome('TXRH',2018)
Error in filter_impl(.data, quo) : Result must have length 727, not 0
```

well, that stinks.

I think I've got enough here to tell me that, moving forward, the edgarWebR/XBRL/finstr pipeline is the way to go.

# A Recap

So, like I said, I have not yet worked out all the kinks involved with iterating through my list of companies with a single swing yet, but the operational flow-chart for the edgardWebR/XBRL/finstr pipeline looks something like this:

1. For an particular company we get a list of all 10-K filings available prior to a specified date
2. For each of those filing we look for the filings details which gives us the location of an .xml file
3. Parse the .xml file
4. Convert the parsed .xml to a financial statement object
5. Save that statement to a list
6. if the routine can't find an xml file or the url request gets timed out, we document the error and save that as a data frame.
7. Repeat steps 2 - 6 for all 10-K filings in the list from Item 1.

At the completion of Steps 1 - 7 we have a list. Call it $$L$$ for now.  That list is a list of lists.  Let's call these elements $$l_{i}$$.  Each $$l_{i}$$ has three elements:

1. The ticker symbol - a character value

2. A list of financial statements - this is itself a list where each list entry corresponds to a 10-K filing

3. A list of errors - this is a list of data frames.  Each data frame in the list contains an error code and a url for which the main routine returned an error.


## A final note on time-out errors

Notice that I retained the list of all links to XBRL INSTANCE DOCUMENTS where the requests got timed out. With this list an enterprising data monkey could pretty easily just write a routine to iterate through this list and retry each url a couple hundred times...or until success.  This would probably require an overnight run but could potentially plug some of the data gaps.





