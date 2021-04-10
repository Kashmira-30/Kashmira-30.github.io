
---
Kashmira De Saram
Lab 6
---
# Creating a Bivariate Map with two vaariables

To begin, you need an api key
- listed below are the packages needed to get tidycensus data
- Set your own workspace
- The place to get census data: https://www.socialexplorer.com/data/metadata/
```{r}
# packages
library(tidyverse)
library(tidycensus)
library(sf)
library(ggplot2)

options(tigris_class = "sf")
options(tigris_use_cache = TRUE)

setwd("C:/Users/Owner/Desktop/GES486_Labs/Lab6")
```
# Data
The two variables I am working with are single female households and single male households from ACS 2019 (5-year estimates). The I specified this data to Montgomery County in Maryland.
```{r}
Mgomery_household_2019 <- get_acs(geography = "tract", 
     variables = c("total_hhtype" = "B11012_001", # All household types
                   "Female_hh" = "B11012_008", # All Female households, no spouse
                   "Male_hh" = "B11012_013", # All Male households, no spouse 
                   "med_hh_inc" = "B19013_001" # Median household income
                   ), 
     year = 2019,
     survey = "acs5",
     state = c(24), 
     county = c(031), 
     geometry = TRUE,
     output = "wide") 
     
```
This will also save the computed data into a csv
```{r}
st_write(Mgomery_household_2014, "Mgomery_household_2014.csv") 
# geometry is false!
```
# Data computation
The following below shows what i analyzed, the first bit of codes shows the percentage of female and male with no spouse households to all other types of households in 2019. And below that is for 2014.
```{r}
# Compute percentage of Female and Male (with np spouse) households in 2019
Mgomery_household_2019$Both_households <- (Mgomery_household_2019$Female_hhE + Mgomery_household_2019$Male_hhE) / Mgomery_household_2019$total_hhtypeE
Mgomery_household_2019$Other_households <- 1- Mgomery_household_2019$Both_household
```
```{r}
# Compute percentage of Female and Male (with np spouse) households in 2014
Mgomery_household_2014$Both_households <- (Mgomery_household_2014$Female_hhE + Mgomery_household_2014$Male_hhE)/ Mgomery_household_2014$total_hhtypeE
Mgomery_household_2014$Other_households <- 1- Mgomery_household_2014$Both_households
```

# Merging tables
To make a bivariate map, we need the data tables to be joined. This shows a right_join of the two tables.
```{r mergeanddiff}
## Merge the two time periods
Mgomery_household <- right_join(Mgomery_household_2019, Mgomery_household_2014, 
                              by="GEOID",
                              suffix=c(".19",".14"))
```

To continue computing the data, this shows the difference in single female households from 2014 to 2019 as well as single male households in 2014 to 2019. Then I used the tranform function to make it into web mercator.
```{r}
# Compute differences in households
Mgomery_household$Female_hhE <- Mgomery_household$Female_hhE.19 -
        Mgomery_household$Female_hhE.14
Mgomery_household$Male_hhE <- Mgomery_household$Male_hhE.19 -
        Mgomery_household$Male_hhE.14
        
Mgomery_household <- st_transform(Mgomery_household, 3857)
```
# To plot the data
I used the st_write function to make the data accesible on QGIS to make the Bivariate maps
```{r}
st_write(Mgomery_household_2019, "Mgomery_household_diff.geojson")
```
