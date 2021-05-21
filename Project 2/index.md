# Quick and simple change over time map using QGIS and R-Studio

##### Necessary programs
* QGIS 
* RStudio
Can download programs through Chocolatey: https://community.chocolatey.org/packages/QGIS
*(this would require you to download Chocolatey first)
* Tutorial on turning maps into webmaps: https://www.qgistutorials.com/en/docs/web_mapping_with_qgis2web.html

## Description
I used Rstudio to get Census Data using an api key. Next I processed that data through Rstudio to find the variables I wanted and then turned it into a geojson file to be edited in QGIS.

## R Code

```{r, message=FALSE, warning=FALSE}
# Packages I used

library(tidyverse)
library(tidycensus)
library(sf)
library(ggplot2)

options(tigris_class = "sf")
options(tigris_use_cache = TRUE)

```
##  Next I'm getting my data from the census tract, I used ACS 5 year estimates (2010-2014) and (2015-2019).
```{r, message=FALSE, warning=FALSE}
# This gets the 2015-2019 population, race+eth, housing units, and median household income from the ACS
# This shows the household type whether its single Female or Male compared to all other types of households

Mgomery_household_2019 <- get_acs(geography = "tract", 
     variables = c("total_hhtype" = "B11012_001", # All household types
                   "Female_hh" = "B11012_008", # All Female households, no spouse
                   "Male_hh" = "B11012_013" # All Male households, no spouse 
                   ), 
     year = 2019,
     survey = "acs5",
     state = c(24), 
     county = c(031), 
     geometry = TRUE,
     output = "wide")
```
```{r, message=FALSE, warning=FALSE}
# This gets the 2010-2014 population, race+eth, housing units, and median household income from the ACS
# This shows the household type whether its single Female or Male compared to all other types of households

Mgomery_household_2014 <- get_acs(geography = "tract", 
     variables = c("total_hhtype" = "B11012_001", # All household types
                   "Female_hh" = "B11012_010", # All Female households, no spouse
                   "Male_hh" = "B11012_007" # All Male households, no spouse 
                   ),      
     year = 2014,
     survey = "acs5",
     state = c(24), 
     county = c(031),
     geometry = TRUE, 
     output = "wide") 
      
```
## Next came the calculating, I've explained what I wanted to analyze with the data I got below.
```{r eveshare, message=FALSE, warning=FALSE}

# Compute Female and Male households
# I took the individual Female and Male households and divided them by the total household types

# Compute percentage of Female and Male (with no spouse) households in 2019
Mgomery_household_2019$Both_households <- (Mgomery_household_2019$Female_hhE + Mgomery_household_2019$Male_hhE) / Mgomery_household_2019$total_hhtypeE

Mgomery_household_2019$Other_households <- 1- Mgomery_household_2019$Both_household

# Compute percentage of Female and Male (with no spouse) households in 2014
Mgomery_household_2014$Both_households <- (Mgomery_household_2014$Female_hhE + Mgomery_household_2014$Male_hhE)/ Mgomery_household_2014$total_hhtypeE

Mgomery_household_2014$Other_households <- 1- Mgomery_household_2014$Both_household


```
## Last step before turning everything into a Geojson was to reproject as a Web-Mercator
```{r, message=FALSE, warning=FALSE}
## Transform data 

Mgomery_household_2019 <- st_transform(Mgomery_household_2019, 3857) 
# reproject into web-mercator 
Mgomery_household_2014 <- st_transform(Mgomery_household_2014, 3857) 
# reproject into web-mercator 

```
```{r, message=FALSE, warning=FALSE}
# Since I've already made them into a Geojson, I'm showing what the code would look like below
## st_write(Mgomery_household_2019, " Mgomery_household_2019.geojson")
## st_write(Mgomery_household_2014, " Mgomery_household_2014.geojson")
```
### Here are the Images I made after symbolizing the data on QGIS
<img src= "Montgomery_County_Households_2010-14.png?raw=true"/>
<img src= "Montgomery_County_Households_2015-19.png?raw=true"/>


