#                                                  Crime Statistics of Robberies in Maryland Counties
### Introduction

  When it comes to crime, there are many kinds and many degrees. This project will look at a type of crime, Robbery, within Maryland’s Counties. To begin, a “robbery” is defined as taking or attempting to take a person’s item of value using force or violence or threatening the victim and putting them in fear. Within Maryland, such crimes are more common in certain counties than others. In the 2015 Crime in Maryland Report, there were more crimes of all sorts reported than in 2014 and crimes such as robberies were up 5.2%. In 2015, there were over 10,000 robberies where 50% were on the street and only 1% was banks. Some of these were further classified as the perpetrator either wielding a fire arm (which was 46%) or personal weapon (which was 37%). Some other demographics included gender and Race, where 89% were male, 11% female, 80% were Black, 20% White, and less than a percent was American Indian and Pacific Asian Islander. These statistics compared to the 2017 report show that robberies increased by another 1,000 but was a 4% decrease from 2016. Also, the racial demographic changed where 77% were by Black and 22% were by White and 1% by American Indian and Pacific Asian Islander, (Crime in Maryland, 2017). For this project, the focus was to compare robberies over MD counties especially in the year 2015. The maps will have also show the average robberies from 2000 to 2017. The project also takes into account the median house hold incomes of each county to census tract and racial background of populations. Other data also includes location and amount of police stations, correctional facilities and schools. Each of these data are put together to see the correlation between the crime statistics of the counties and how each factor could correlate with the crime rate of each county.
  
  Personally, I wanted to look at these crime statistics because they’re things I would see reported on the media constantly. I wanted to look at the crime rate of counties with more minorities and compare the resources that are available, like schools. I also wanted to see if there is a connection between having a certain number of police stations and correctional facilities in a county and the crime rate. I’ve made many maps that will help to understand the different reasons as to why crime rates pertaining to robberies might be higher in some counties than others. This project will also use assumptions and see if the data correlates with them or not.

### First steps
1)	Download crime statistic CSV, Maryland state boundaries data, Police stations, correctional facilities, and schools.
2)	Join the CSV data with the boundary shapefile using plugin MMQGIS -> Combine -> attributes join by CSV file, in this case I had to individually change the name type of the fields on the CSV to integer using the refractor field tool.
3)	Use Toggle editing mode on the new layer and filter out all years before 2000. Do this by going to open attribute table -> select features by expression -> “Year” < “2000”  
4)	Add all the different point data (the stations and facilities).
5)	Make many layers so different analyses can be done with the same data

### Analysis
#### Make change over time map:
To make the change over time map, I focused on every 5 years (2000, 2005,…etc.) to 2017. I filtered out all other years by going to select by attribute and using the field “Year” = 2015. This highlighted all the Counties with data on year 2015. By inverting the selection, I deleted all the other years so I would have 2015 left. I made several layers from the original boundary layer that I joined with the CSV data and saved them as a Geojson before making any layers for individual years. Finally, after I made all the years I wanted to showcase, I used the “graduated” symbology to show the Robberies in each county for MD, I used 10 classes and Quantile as my classification. With these maps, I made a gif to show how the number of robberies changed every 5 years and what counties had an increase or decrease.

#### Making a median household income map with total population (Bivariate Map):
This was done in RStudio
The point of this map was to see how the population of an area correlates with Median Household income in each tract. This is to see if the areas of higher crime are areas of higher or lower income and higher or lower total population.

#### Here is the process I used
---------------------------------------------------------------------------------------
```{r, message=FALSE, warning=FALSE}
knitr::opts_knit$set(root.dir = "C:/Users/Owner/Desktop/Final_project") 
library(tidycensus)
library(tidyverse)
library(sf)
library(geojson)
library(readr)
library(ggplot2)
library(sp) 
library(scales) 
library(dplyr)

options(tigris_class = "sf")
options(tigris_use_cache = TRUE)

MD_county_mhh4 <- get_acs(geography = "tract", variables = c("Med_hh_inc" = "B19013_001", "Total_pop" = "B01003_001"),
                           year = 2015, 
                           survey = "acs5", 
                           state = c(24), 
                           geometry = TRUE,  
                           output = "wide")

library(biscale)
library(cowplot)
library(leaflet)


MD_county_mhh4_filt <- MD_county_mhh4 %>%
  filter(Med_hh_incE > 0)

MD_county_mhh5 <- bi_class(MD_county_mhh4_filt, x = Total_popE, y = Med_hh_incE, style = "quantile", dim = 3)

windowsFonts("Arial" = windowsFont("Arial"))

map1 <- ggplot() + geom_sf(data = MD_county_mhh5, mapping = aes(fill = bi_class), color = "white", size = NA, show.legend = FALSE) +  bi_scale_fill(pal = "DkBlue", dim = 3)  + labs(title = "Population and Median Household Income", subtitle = "Dark Blue (DkBlue) Palette") + bi_theme() + theme(text = element_text(family = "Arial"), plot.title = element_text(size = 16), plot.subtitle = element_text(size = 10))

Legend1 <- bi_legend(pal = "DkBlue", 
                     dim = 3,
                     xlab = "Higher Total Population ", 
                     ylab = "Higher Med Income ", size = 4) 
ggdraw() + 
  draw_plot(map1, 0, 0, 1, 1) +
  draw_plot(Legend1, 0.2, 0.02, 0.2, 0.2)
```
---------------------------------------------------------------------------------------
#### Final output
<img src= "Bivariate_mhhi.png?raw=true"/>



#### Making racial demographic maps for Black, White, and Asian populations in MD:
This was another RStudio map, using census data to show the demographic.

#### Process:
---------------------------------------------------------------------------------------

```{r, message=FALSE, warning=FALSE}
knitr::opts_knit$set(root.dir = "C:/Users/Owner/Desktop/Final_project") 
library(tidycensus)
library(tidyverse)
library(sf)
library(geojson)
library(readr)
library(ggplot2)
library(sp) 
library(scales) 
library(janitor) 
library(readr)
library(dplyr)

options(tigris_class = "sf")
options(tigris_use_cache = TRUE)

MD_county_mhh3 <- get_acs(geography = "tract", variables = c("Med_hh_inc" = "B19013_001", "White_pop" = "B02001_002", "Black_pop" = "B02001_003", "Asian_pop" = "B02001_005"),
                           year = 2015, 
                           survey = "acs5", 
                           state = c(24), 
                           geometry = TRUE,  
                           output = "wide")

ggplot(MD_county_mhh3) + geom_sf(aes(fill = White_popE, colour = White_popE))
ggplot(MD_county_mhh3) + geom_sf(aes(fill = Black_popE, color = Black_popE))
ggplot(MD_county_mhh3) + geom_sf(aes(fill = Asian_popE, color = Asian_popE))
```
----------------------------------------------------------------------------------------
#### Final Output
<img src= "white_pop.png?raw=true"/>        <img src= "Black_pop.png?raw=true"/>         <img src= "Asian_pop.png?raw=true"/>   

#### Making an average robberies (2000 -2017) map:

To Make this map, I made another layer from the joined layer and saved it as a Geojson before making any changes. Then I re-added it and made a new field where I calculated the average of Robberies in each county for 18 years. To make a new field, I went to the attribute table -> Toggle editing -> field calculator. Here I used the “create new field” option and the named it, and used the expression mean(“Robberies” , ”counties”). This filled in the field with the average robberies of each county. I then used graduated symbology and used 10 classes with Quantile with my classification. I also added all the police stations and Facilities to this map and changed their symbology to match the state and to differentiate them. Then I used the nearest neighbor function to connect the county police stations to the local correctional facilities. With this data I was able to find that Baltimore county and Baltimore City had the highest connections from the local facilities to county police. Baltimore County and City also had the largest amount of county police stations from all other counties. Interestingly, the greatest average amount of Robberies has also been with Baltimore County, City, and other Counties such as Montgomery and Prince George’s. Also, comparing the racial demographics from the last maps to this show that there is separation between different racial groups. While the White population is more scattered, the Black population is more concentrated in Baltimore City to central Maryland, and Asian populations mainly in central Maryland. The next map which shows the estimates of median household income show that that Baltimore City is one of the lower income areas of MD.

#### Final Output

<img src= "Final_Map1.png?raw=true"/>         <img src= "Final_Map1_legend.png?raw=true"/> 

#### Making a Median Household Income Map (5-year estimate 2015):
I made another map getting the data from the ACS 5-year estimate to showcase lower to higher median income by state.

 <img src= "Med_hh_Income.png?raw=true"/> 

The Last Map I did was to compare the number of Schools in each county and compare that to the robbery rate per 100,000 people in 2015.

 <img src= "schools_counties.png?raw=true"/>        <img src= "schools_counties_p.png?raw=true"/> 
 
#### Robbery rates per 100,000 population:

<img src= "Map_2015.png?raw=true"/> 

First, I made the added the schools and categorized the by type in symbology. Then by using the count points in polygon tool, I was able to make a map to show which counties had the most educational facilities. Interestingly, some areas with higher robbery rates also have the higher number of educational facilities not just counties with lower robbery rates. This was different from my assumption that higher crime would happen in areas with less access to education. But once I analyzed the population of each area, it made sense that these counties would have more facilities as they have higher populations.

I realized this could be made into a Bivariate map for better explanation, so this is the Final Map.

<img src= "Bivariate_last.png?raw=true"/> 

### Conclusion
Throughout this project, several maps were made to analyze crime statistics of robbery to other variables. The maps give a lot of insight into the demographics of the counties with higher crime rates when it comes to Robberies. Some interesting things I’ve found from this project is that not all assumptions are valid. In some counties with lower crime rates in terms of robbery, there is less educational facilities than that of counties of higher crime rates. There are also more police stations in counties of higher crime rates in terms of robbery than those with lower. This is interesting because we would assume with more police stations, there would be less crime, but it could also be because there is more crime, there is a need for more police stations. What I also found interesting is that each state has a correctional facility that is either next to a police station or is near a police station. Another interesting part of the data was that Baltimore City consistently shows more crime in terms of robbery but also has more educational facilities. The city also has a lower median household income than its surrounding counties. There is a trend that central Maryland has higher median household income and educational facilities as well as higher populations. The racial demographics also show that central Maryland also has higher minority populations (Asian and Black populations form graphs) and a high overall population which could explain higher robbery rates in some of the counties. Since this data is focused on mainly 2015 to 2017, the amount of crime could also be related to social issues that have happened during those years. In all, I think this research could go further in terms looking at schools to see if they allow students to grow in healthy environments.  Looking at families and types of families especially in areas of higher crime and on how a certain background could correlate with crime in terms of robbery. There are many other variables that might be a reason to the higher rates of robberies which could be further explored.

### Data Sources:
* Maryland County Boundary: https://data.imap.maryland.gov/datasets/4c172f80b626490ea2cff7b699febedb_1?geometry=-80.737%2C38.076%2C-73.799%2C39.574
* Federal Police Stations: https://data.imap.maryland.gov/datasets/maryland-police-federal-police-stations?geometry=-77.337%2C38.926%2C-76.470%2C39.113
* Maryland County Police Stations: https://data.imap.maryland.gov/datasets/maryland-police-county-police-stations?geometry=-80.869%2C38.138%2C-73.931%2C39.635
* Maryland Local Correctional Facilities: https://data.imap.maryland.gov/datasets/maryland-correctional-facilities-local-correctional-facilities?geometry=-80.860%2C38.149%2C-73.922%2C39.645
* Maryland State Correctional facilities: https://data.imap.maryland.gov/datasets/maryland-correctional-facilities-state-correctional-facilities?geometry=-80.726%2C38.132%2C-73.788%2C39.629
* Maryland K-12 Public schools: https://data.imap.maryland.gov/datasets/f49c4bb1a9a74029ae974e6d6fd08b71_5?geometry=-80.755%2C38.097%2C-73.818%2C39.594
* Crime data: https://opendata.maryland.gov/Public-Safety/Violent-Crime-Property-Crime-by-County-1975-to-Pre/jwfa-fdxs
* Maryland Department of State Police (2015). Crime in Maryland: Uniform Crime Report: UCR [Annapolis, MD]: Maryland Depart. of State Police
* Maryland Department of State Police (2017). Crime in Maryland: Uniform Crime Report: UCR [Annapolis, MD]: Maryland Depart. of State Police










