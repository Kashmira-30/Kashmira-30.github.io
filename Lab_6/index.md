# Creating a Bivariate Map with Tidycensus Data

#### Procedure
- will need an api key
##### (packages) 
library(tidycensus)
library(tidyverse)

# packages
library(tidyverse)
library(tidycensus)
library(sf)
library(ggplot2)


options(tigris_class = "sf")
options(tigris_use_cache = TRUE)


#### Data
- Get from (https://www.socialexplorer.com/data/metadata/)
- I used the Single Female and Male households census data with ACS 2019 (5-year estimates)
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






<img src="Bivariate_map.png?raw=true"/>
