
In Part 1 of this series I covered a ghetto web-crawling route used to pull information from 10-K financial disclosures from the SEC's EDGAR database.  That solution relied on the R package [finreportr](https://cran.r-project.org/web/packages/finreportr/index.html) which gets points for being very easy to use but loses points for being a little unpredicatable in terms of what data it actually returns you.

The solution I'll talk about here uses the following approach:

1. Use functions from the [edgarWebR](https://github.com/mwaldstein/edgarWebR) library to get details on available 10-K filings for a company.
2. Use the [XBRL](https://cran.r-project.org/web/packages/XBRL/index.html) library to parse the .xml documents found in Step 1 above.
3. Use functions in the [finstr](https://github.com/bergant/finstr) library to further parse data retreived into objects of class 'financial statement'




## Quick Walk Through

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



