

## A Quick California Insurance Example

This exercise was shamelessly pirated from [Kevin Johnson's badass post](http://www.kevjohnson.org/making-maps-in-r/). 

As a proof-of-concept to verify that I could actually read-in and manipulate shapefiles in R I tried to reproduce Kevin's map of insured vs. uninsured individuals by Census Tract.  I made one tiny change: I used data for California instead of Georgia.

I got shapefiles for Census Tract Boundaries and California County Boundaries from the Cartographic Boundaries aisle of the Census Bureau's TIGER Products web-mart.  You can download the shapefiles 

[here](http://www.census.gov/geo/maps-data/data/tiger-cart-boundary.html)  

Or you can get them from the GitHub Repo I set up

[Aaron's R-spatial Repo](https://github.com/aaronmams/R-spatial.git)

In addition to county boundaries, I grabbed the estimated number of individuals with health insurance in California by Census Tract.  This was a bit of a pain in the ass because the Census Bureau's American Community Survey is notoriously difficult to navigate.   


![CA insurance](images/CA_insurance.png)
