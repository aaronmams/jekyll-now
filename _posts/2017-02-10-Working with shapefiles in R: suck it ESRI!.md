

## A Quick California Insurance Example

This exercise was shamelessly pirated from [Kevin Johnson's badass post](http://www.kevjohnson.org/making-maps-in-r/). 

As a proof-of-concept to verify that I could actually read-in and manipulate shapefiles in R I tried to reproduce Kevin's map of insured vs. uninsured individuals by Census Tract.  I made one tiny change: I used data for California instead of Georgia.

I got shapefiles for Census Tract Boundaries and California County Boundaries from the Cartographic Boundaries aisle of the Census Bureau's TIGER Products web-mart.  You can download the shapefiles 

[here](http://www.census.gov/geo/maps-data/data/tiger-cart-boundary.html)  

Or you can get them from the GitHub Repo I set up

[Aaron's R-spatial Repo](https://github.com/aaronmams/R-spatial.git)

In addition to county boundaries, I grabbed the estimated number of individuals with health insurance in California by Census Tract.  This was a bit of a pain in the ass because the Census Bureau's American Community Survey is notoriously difficult to navigate.  You can try to get this series yourself if you want from the [ACS website](https://factfinder.census.gov/faces/nav/jsf/pages/guided_search.xhtml): to keep things simple I used Series B27001A, "Health Insurance Coverage Status by Age (Whites Only)" and I used the spatial resolution of Census Tract. I also have this data in a cleaned form in a .csv file in the GitHub repo.  I suggest using my clean version first then branching out to other data files later.   

```R
library(rgdal)
library(ggplot2)
library(ggmap)
library(scales)
library(maptools)
library(rgeos)
library(dplyr)

#Read the Census Tract Boundaries for California
# note dsn is just the directory where the shapefile you want lives
tract <- readOGR(dsn = "data/gz_2010_06_140_00_500k", 
                 layer = "gz_2010_06_140_00_500k")
tract <- fortify(tract, region="GEO_ID")

#Read in the .csv file with insured/uninsured individuals
ins <- read.csv("data/CA_insured_tract.csv")
ins$id <- as.character(ins$GEO.id)
ins$percent <- ins$Insured18_64/ins$Pop18_64

#clip the data and the shapefile boundaries
plotData <- left_join(tract, ins)

#A first plot
ggplot() +
  geom_polygon(data = plotData, aes(x = long, y = lat, group = group,
                                    fill = percent), color = "black", size = 0.25) + 
        theme_bw()

#-------------------------------------------------------------
#Now let's overlay county boundaries instead of census tract 
# boundaries since the census tract boundaries are kind of 
# bleeding together and making shit hard to see
county <- readOGR(dsn = "data/gz_2010_06_060_00_500k", 
                  layer = "gz_2010_06_060_00_500k")
county <- fortify(county, region="COUNTY")

p <- ggplot() +
  geom_polygon(data = plotData, aes(x = long, y = lat, group = group,
                                    fill = percent)) +
  geom_polygon(data = county, aes(x = long, y = lat, group = group),
               fill = NA, color = "black", size = 0.25) +
  coord_map() + theme_bw()

#now I'm going to use the scales() package to bin the data a little
# so the contrasts in coverage rates are a little more visible
ggplot() +
  geom_polygon(data = plotData, aes(x = long, y = lat, group = group,
                                    fill = percent)) +
  geom_polygon(data = county, aes(x = long, y = lat, group = group),
               fill = NA, color = "black", size = 0.25) +
  coord_map() +
  scale_fill_distiller(palette = "Greens", labels = percent,
                       breaks = pretty_breaks(n = 10)) +
  guides(fill = guide_legend(reverse = TRUE)) + theme_bw()
#---------------------------------------------------------------

```


![CA insurance](/images/CA_insured.png)
