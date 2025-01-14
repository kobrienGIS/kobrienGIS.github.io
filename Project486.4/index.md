# Mapping and Predicting Gentrification in Washington D.C. from 2010 to 2019

<img style="border:1px solid black;" src="GentInterpFinalMap.svg?raw=true"/>

## Project Description:
This project was a multi-map time series analysis with several census variable indicators of gentrification in Washington DC census tracts. Each indicator was mapped in a change map from 2010 to 2019 and each indicator was eventually factored into an overall gentrification risk map. The risk map was also then interpolated to expand spatial predictions of gentrification risk levels to the rest of Washington DC. Most of the project was completed in RStudio, however the interpolated map needed to be finished in QGIS and the map layouts were finalized in Inkscape.

## Project Goal:
Use tidycensus and variables from the 2010 and 2019 ACS surveys to map changes in indicators of gentrification over time to assess and predict the risk of gentrification across census tracts in Washington DC.  

## Project Plan:
- Map of change from 2010 to 2019 in % of Bachelors Degrees
- Map of change from 2010 to 2019 in % Non-White Residents
- Map of change from 2010 to 2019 in Median Age
- Map of change from 2010 to 2019 in Per Capita Income
- Map of change from 2010 to 2019 in Median Household Income
- Map of change from 2010 to 2019 in Median Home Value
- Map that synthesizes variables into a Gentrification risk map, using threshholds like percentiles and arbitrary limits set in prior literature
- Map that uses centroids from the Gentrification risk map and then interpolates to create a raster or substitute geometry layer for predictions

## Map 1: Education

<img style="border:1px solid black;" src="BachFinalMap.svg?raw=true"/>

Observations: <br>
From 2010 to 2019 we can see that the percentage of residents with at least a bachelors degree increased across most areas in DC. Particularly noticeable are several census tracts in the center that experienced sharp increases (30–50%) in residents with college education, possibly indicating gentrification. 

## Map 2: Race 

<img style="border:1px solid black;" src="NWFinalMap.svg?raw=true"/>

Observations: <br>
From 2010 to 2019 we can see here that the percentage of non-white residents decreased across most census tracts in the center to the northeast of DC, while percentages rose slightly in the west. Another strong indicator of gentrification in DC is displacement of minorities, this shows that many non-white individuals and families left central DC (generally 20–40%) over the 9 year timeframe.  

## Map 3: Age

<img style="border:1px solid black;" src="AgeFinalMap.svg?raw=true"/>

Observations: <br>
Here we see that between 2010 and 2019 there was a large range of changes in median age of residents across DC. Notably, many central and eastern census tracts experienced sharp decreases in median age, while more tracts around the periphery experienced sharp rises. As for gentrification, this data is inconclusive. 

## Map 4: Personal Income

<img style="border:1px solid black;" src="IncomeFinalMap.svg?raw=true"/>

Observations: <br>
Per capita income largely increased across DC, as is normally expected. However, several tracts in the city's center experienced sharp increases of upwards of $40,000, potentially indicating a positive shift in residents with high income in these areas. 

## Map 5: Household Income

<img style="border:1px solid black;" src="MHIFinalMap.svg?raw=true"/>

Observations: <br>
Median household income (MHI) is often regarded as one of the most important indicators of gentrification. Here we can see that several tracts experienced sharp increases in MHI upwards of $60,000, most likely indicating areas that have recently attracted high-income families.

## Map 6: Home Value

<img style="border:1px solid black;" src="ValueFinalMap.svg?raw=true"/>

Observations: <br>
As the final variable included in this analysis, median home value provides another useful indicator of gentrification. Several tracts in the city center can be seen having large increases in median home value upwards of $300,000. Note, this data range has a few outliers. 

## Map 7: Gentrification Risk

<img style="border:1px solid black;" src="GentFinalMap.svg?raw=true"/>

Observations: <br>
After computing each of the variables into a risk-graded classification, several tracts in the center appear to have experienced gentrification based on the previous indicators. Overall, gentrification appears to be concentrated in the center of DC and in the census tracts surrounding Capitol Hill. One thing to note is that there were several areas with potential false positives, particularly in the west. 

## Map 8: Interpolated Gentrification

<img style="border:1px solid black;" src="GentInterpFinalMap.svg?raw=true"/>

## Main Takeaways:

- Several tracts in the central portions of Washington DC likely experienced significant degrees of gentrification
- Most of the indicators contributed in some way to our understanding of gentrification in Washington DC
- Overall, wealth increased across Washington DC, although some areas experienced much sharper increases in wealth than others
- Washington DC attracted more college-educated and higher-income individuals resulting in economic growth likely as a result of gentrification
- Washington DC experienced significant levels of racial displacement likely as a result of gentrification

Our analysis of known indicators of gentrification in Washington DC from 2010 to 2019 produced a range of results consistent with earlier studies of gentrification. All of the indicators predicted at least some degree of gentrification across the 9-year period. When combined, the output risk classification map yielded a useful visualization of where gentrification most likely occurred. The IDW interpolation analysis added to this by producing a spatial prediction map that expanded these findings across census tracts in a more natural manner. Predictive models like the one this analysis produced may prove instrumental in estimating and assessing gentrification in the future. 

## Sources and References
Data: <br>
- U.S. Census Bureau 5-year American Community Surveys for 2010 and 2019

Literature:
- Lang, Marissa J. “Gentrification in D.C. Means Widespread Displacement, Study Finds.” The Washington Post, WP Company, 27 Apr. 2019, https://www.washingtonpost.com/local/in-the-district-gentrification-means-widespread-displacement-report-says/2019/04/26/950a0c00-6775-11e9-8985-4cf30147bdca_story.html. 
- Preis, Benjamin, et al. “Mapping Gentrification and Displacement Pressure: An Exploration of Four Distinct Methodologies.” Urban Studies (Sage Publications, Ltd.), vol. 58, no. 2, Feb. 2021, pp. 405–24. EBSCOhost, https://doi.org/10.1177/0042098020903011.
- Richardson, Jason, et al. “Shifting Neighborhoods: Gentrification and Cultural Displacement in American Cities.” NCRC, 8 Mar. 2022, https://ncrc.org/gentrification/. 
- “Detecting Gentrification in Washington DC.” RPubs, Tidy Insights, https://api.rpubs.com/Tidy_Insights/fgproject. 

---

## Project Script:
title: "Gentrification" <br>
author: "Kyle O'Brien" <br>
date: '2022-05-20'

Packages
```r
library(tidycensus)
library(tidyverse)
library(dplyr)
library(sf)
library(tigris)
library(rgdal)
library(terra) 
library(RColorBrewer)
library(tidyr)
library(gstat)
library(sp)
library(raster)
library(maptools)
```

Data generation with censusapi calls
```r
# Variables for 2010, start of time-series
vars2010 <- get_acs(
  geography = "tract",
  state = 11,
  variables = c("edu"=c("B15002_001", "B15002_015",
                        "B15002_016", "B15002_017",
                        "B15002_018", "B15002_032",
                        "B15002_033", "B15002_034",
                        "B15002_035"),
                "race"=c("B03002_001", "B03002_003"),
                "age"="B01002_001",
                "income"="B19301_001",
                "mhi"="B19013_001",
                "value"="B25077_001"),
  output = "wide",
  year = 2010,
  geometry = TRUE
) 

# Variables for 2019, end of time-series
vars2019 <- get_acs(
  geography = "tract",
  state = 11,
  variables = c("edu"=c("B15003_001", "B15003_022",
                        "B15003_023", "B15003_024",
                        "B15003_025"),
                "race"=c("B03002_001", "B03002_003"),
                "age"="B01002_001",
                "income"="B19301_001",
                "mhi"="B19013_001",
                "value"="B25077_001"),
  output = "wide",
  year = 2019
)

# Joins the data from 2010 and 2019 for comparison
varsJoined = vars2010 %>% left_join(vars2019, by = "GEOID")
```

Raw data export
```r
st_write(vars2010, "vars2010.geojson")
st_write(vars2019, "vars2019.geojson")
st_write(varsJoined, "varsJoined.geojson")
```

Initial data prep and calculations
```r
# Combines variables for education to create a column with at least bachelors degrees and then calculates a percentage
# 2010 Bachelors
varsJoined <- varsJoined %>% mutate(bach2010 = edu2E.x + edu3E.x + edu4E.x + edu5E.x + edu6E + edu7E + edu8E + edu9E)
varsJoined <- varsJoined %>% mutate(bachper2010 = bach2010 / edu1E.x)
# 2019 Bachelors
varsJoined <- varsJoined %>% mutate(bach2019 = edu2E.y + edu3E.y + edu4E.y + edu5E.y)
varsJoined <- varsJoined %>% mutate(bachper2019 = bach2019 / edu1E.y)
# Difference in Percentage of Residents with Bachelors Degrees
varsJoined <- varsJoined %>% mutate(bachdiff = bachper2019 - bachper2010)
  
# Calculates the percentage of non-hispanic white residents per tract and then takes the inverse to get the percentage of non-white residents
# 2010 Non-White
varsJoined <- varsJoined %>% mutate(whiteper2010 = race2E.x / race1E.x)
varsJoined <- varsJoined %>% mutate(nonwhiteper2010 = 1 - whiteper2010)
# 2019 Non-White
varsJoined <- varsJoined %>% mutate(whiteper2019 = race2E.y / race1E.y)
varsJoined <- varsJoined %>% mutate(nonwhiteper2019 = 1 - whiteper2019)
# Difference in Percentage of Non-White Residents
varsJoined <- varsJoined %>% mutate(nonwhitediff = nonwhiteper2019 - nonwhiteper2010)

# Differences in Median Age
varsJoined <- varsJoined %>% mutate(agediff = ageE.y - ageE.x)

# Differences in Per Capita Income
varsJoined <- varsJoined %>% mutate(incomediff = incomeE.y - incomeE.x)

# Differences in Median Household Income
varsJoined <- varsJoined %>% mutate(mhidiff = mhiE.y - mhiE.x)

# Differences in Median Home Value
varsJoined <- varsJoined %>% mutate(valuediff = valueE.y - valueE.x)
```

### Functions

Map Visualization
```r
EduMap <- function(dataset) {
  # Visualizes the map with a divergent color palette and gives it a title and legend
  eduproduct <- ggplot(dataset) + 
      geom_sf(aes(fill = bachdiff), color = alpha("black", 1 / 2), size = 0.25) + 
      scale_fill_distiller(palette = "PuOr", direction = 1, guide = "colorbar", limits = c(-.50, .50), 
                           breaks = c(-.40,-.30,-.20,-.10, 0, .10, .20, .30, .40), 
                           labels = c("-40","-30","-20","-10", "No Change", "10", "20", "30", "40")) +
      theme_void() + 
      theme(text = element_text(size = 12, family = "mono")) +
      theme(plot.title = element_text(hjust = 0.5)) +
      theme(legend.key.height = unit(1, 'cm')) +
      labs(fill = "Difference in % of \nBachelors Degrees",
          title = "College Education in Washington DC \nfrom 2010 to 2019")
  
  # Returns the final map product for export
  return(eduproduct)
}
  
RaceMap <- function(dataset) {
  # Visualizes the map with a divergent color palette and gives it a title and legend
  raceproduct <- ggplot(dataset) + 
      geom_sf(aes(fill = nonwhitediff), color = alpha("black", 1 / 2), size = 0.25) + 
      scale_fill_distiller(palette = "PuOr", direction = 1, guide = "colorbar", limits = c(-.50, .50), 
                           breaks = c(-.40,-.30,-.20,-.10, 0, .10, .20, .30, .40), 
                           labels = c("-40","-30","-20","-10", "No Change", "10", "20", "30", "40")) +
      theme_void() + 
      theme(text = element_text(size = 12, family = "mono")) +
      theme(plot.title = element_text(hjust = 0.5)) +
      theme(legend.key.height = unit(1, 'cm')) +
      labs(fill = "Difference in % of \nNon-White Residents",
          title = "Racial Displacement in Washington DC \nfrom 2010 to 2019")
  
  # Returns the final map product for export
  return(raceproduct)
}

AgeMap <- function(dataset) {
  # Visualizes the map with a divergent color palette and gives it a title and legend
  ageproduct <- ggplot(dataset) + 
      geom_sf(aes(fill = agediff), color = alpha("black", 1 / 2), size = 0.25) + 
      scale_fill_distiller(palette = "PuOr", direction = 1, guide = "colorbar", limit = c(-20, 20), 
                           breaks = c(-15,-10,-5, 0, 5, 10, 15),
                           labels = c("-15","-10","-5","No Change", "5", "10", "15")) +
      theme_void() + 
      theme(text = element_text(size = 12, family = "mono")) +
      theme(plot.title = element_text(hjust = 0.5)) +
      theme(legend.key.height = unit(1, 'cm')) +
      labs(fill = "Difference in Median \nAge (in years)",
          title = "Median Age of Residents in \nWashington DC from 2010 to 2019")
  
  # Returns the final map product for export
  return(ageproduct)
}

IncomeMap <- function(dataset) {
  # Visualizes the map with a divergent color palette and gives it a title and legend
  incomeproduct <- ggplot(dataset) + 
      geom_sf(aes(fill = incomediff), color = alpha("black", 1 / 2), size = 0.25) + 
      scale_fill_distiller(palette = "PuOr", direction = 1, guide = "colorbar", limits = c(-60000, 60000), 
                           breaks = c(-50000,-40000,-30000,-20000,-10000, 0, 10000, 20000, 30000, 40000, 50000), 
                           labels = c("-50,000","-40,000","-30,000","-20,000","-10,000", "No Change", "10,000", "20,000", 
                                      "30,000", "40,000", "50,000")) +
      theme_void() + 
      theme(text = element_text(size = 12, family = "mono")) +
      theme(plot.title = element_text(hjust = 0.5)) +
      theme(legend.key.height = unit(1, 'cm')) +
      labs(fill = "Difference in Per \nCapita Income (in USD)",
          title = "Per Capita Income in Washington DC \nfrom 2010 to 2019")
  
  # Returns the final map product for export
  return(incomeproduct)
}

MhiMap <- function(dataset) {
  # Visualizes the map with a divergent color palette and gives it a title and legend
  mhiproduct <- ggplot(dataset) + 
      geom_sf(aes(fill = mhidiff), color = alpha("black", 1 / 2), size = 0.25) + 
      scale_fill_distiller(palette = "PuOr", direction = 1, guide = "colorbar", limits = c(-100000, 100000), 
                           breaks = c(-80000,-60000,-40000,-20000, 0, 20000, 40000, 60000, 80000), 
                           labels = c("-80,000","-60,000","-40,000","-20,000", "No Change", "20,000", "40,000", "60,000", "80,000")) +
      theme_void() + 
      theme(text = element_text(size = 12, family = "mono")) +
      theme(plot.title = element_text(hjust = 0.5)) +
      theme(legend.key.height = unit(1, 'cm')) +
      labs(fill = "Difference in \nMHI (in USD)",
          title = "Median Household Income (MHI) in \nWashington DC from 2010 to 2019")
  
  # Returns the final map product for export
  return(mhiproduct)
}

ValueMap <- function(dataset) {
  # Visualizes the map with a divergent color palette and gives it a title and legend
  valueproduct <- ggplot(dataset) + 
      geom_sf(aes(fill = valuediff), color = alpha("black", 1 / 2), size = 0.25) + 
      scale_fill_distiller(palette = "PuOr", direction = 1, guide = "colorbar", limit = c(-600000, 600000), 
                           breaks = c(-500000,-400000,-300000,-200000,-100000, 0, 100000, 200000, 300000, 400000, 500000),
                           labels = c("-500,000","-400,000","-300,000","-200,000","-100,000", "No Change", "100,000", "200,000", 
                                      "300,000", "400,000", "500,000")) +
      theme_void() + 
      theme(text = element_text(size = 12, family = "mono")) +
      theme(plot.title = element_text(hjust = 0.5)) +
      theme(legend.key.height = unit(1, 'cm')) +
      labs(fill = "Difference in Median \nHome Value (in USD)",
          title = "Median Home Value in Washington DC \nfrom 2010 to 2019")
  
  # Returns the final map product for export
  return(valueproduct)
}


# Modifies the dataset for use in the final two maps
GentData <- function(dataset) {
  # Copies the dataset and drops all null values
  gentvars <- dataset
  gentvars[is.na(gentvars)] <- 0

  # Determines thresholds based on percentiles of the data and assigns class values
  # Bachelors Degrees
  quantile(gentvars$bachdiff, probs = c(.60, .75, .90))
  #       60%       75%       90% 
  # 0.1234288 0.1818144 0.2672396 
  gentvars <- gentvars %>% 
    mutate(bachclass = as.numeric(case_when(bachdiff >= 0.2672396 ~ "3",
                                 bachdiff >= 0.1818144 ~ "2",
                                 bachdiff >= 0.1234288 ~ "1",
                                 TRUE ~ "0")))
  # Non White Residents
  quantile(gentvars$nonwhitediff, probs = c(.10, .25, .40))
  #         10%         25%         40% 
  # -0.21654861 -0.13507000 -0.07031094
  gentvars <- gentvars %>% 
    mutate(nonwhiteclass = as.numeric(case_when(nonwhitediff <= -0.21654861 ~ "3",
                                 nonwhitediff <= -0.13507000 ~ "2",
                                 nonwhitediff <= -0.07031094 ~ "1",
                                 TRUE ~ "0")))
  # Median Age
  quantile(gentvars$agediff, probs = c(.10, .25, .40)) 
  #   10%   25%   40% 
  # -6.96 -3.70 -1.30 
  gentvars <- gentvars %>% 
    mutate(ageclass = as.numeric(case_when(agediff <= -6.96 ~ "3",
                                 agediff <= -3.70 ~ "2",
                                 agediff <= -1.30 ~ "1",
                                 TRUE ~ "0")))
  # Per Capita Income
  quantile(gentvars$incomediff, probs = c(.60, .75, .90))
  #     60%     75%     90% 
  # 16223.8 21704.0 30080.8  
  gentvars <- gentvars %>% 
    mutate(incomeclass = as.numeric(case_when(incomediff >= 30080.8 ~ "3",
                                 incomediff >= 21704.0 ~ "2",
                                 incomediff >= 16223.8 ~ "1",
                                 TRUE ~ "0")))
  # Median Household Income
  quantile(gentvars$mhidiff, probs = c(.60, .75, .90))
  #     60%     75%     90% 
  # 34549.0 43710.0 54770.6  
  gentvars <- gentvars %>% 
    mutate(mhiclass = as.numeric(case_when(mhidiff >= 54770.6 ~ "3",
                                 mhidiff >= 43710.0 ~ "2",
                                 mhidiff >= 34549.0 ~ "1",
                                 TRUE ~ "0")))
  # Median Home Value
  quantile(gentvars$valuediff, probs = c(.60, .75, .90))
  #    60%    75%    90% 
  # 149480 196200 255260  
  gentvars <- gentvars %>% 
    mutate(valueclass = as.numeric(case_when(valuediff >= 255260 ~ "3",
                                 valuediff >= 196200 ~ "2",
                                 valuediff >= 149480 ~ "1",
                                 TRUE ~ "0")))

  # Adds each class variable together and uses thresholds to create an overall classification scheme
  gentvars <- gentvars %>% 
    mutate(gentclassnums = bachclass + nonwhiteclass + ageclass + incomeclass + mhiclass + valueclass)
  gentvars <- gentvars %>% 
    mutate(gentclasses = case_when(gentclassnums >= 14.4 ~ "High Risk",
                                   gentclassnums >= 10.8 ~ "Moderately High Risk",
                                   gentclassnums >= 7.2 ~ "Moderate Risk",
                                   gentclassnums >= 3.6 ~ "Low Risk",
                                   TRUE ~ "No Risk"))
  
  # Returns the edited gentrification data
  return(gentvars)
}


# Classified gentrification risk map based on values of previous maps
GentMap <- function(dataset) {
  risk_palette = c("High Risk" = "#f03b20", "Moderately High Risk" = "#feb24c", "Moderate Risk" = "#ffeda0", 
                   "Low Risk" = "#addd8e", "No Risk" = "#f7f7f7")
  
  # Visualizes the data in a risk map
  gentriproduct <- ggplot(dataset) + 
    geom_sf(aes(fill = gentclasses), color = alpha("black", 0)) + 
    scale_fill_manual(values = risk_palette) +
    theme_void() +
    theme(text = element_text(size = 12, family = "mono")) +
    theme(plot.title = element_text(hjust = 0.5)) +
    theme(legend.key.height = unit(1, 'cm')) +
    labs(fill = "Gentrification Risk",
         title = "Gentrification in Washington DC \nfrom 2010 to 2019") +
    geom_sf(fill = NA, color = alpha("black", 1 / 2), size = 0.25)

  # Returns the final map product for export
  return(gentriproduct)
}


# (Currently Not Functional in RStudio)
# Interpolated map based on centroids of areas assessed to be at risk of gentrification
InterpMap <- function(dataset) {
  # Could not find a solution in R, temporarily solved using QGIS
  
  # Returns the final map product for export
  return(interpolproduct)
}
```

Map export
```r
ExportMaps <- function(mapproduct, filename) {
   # Exports each map. The default path is set to the working directory.
   ggsave(
      filename,
      plot = mapproduct,
      scale = 1,
      dpi = 300,
      bg = "white"
   )
}
```

Main code
```r
# Displays and Exports the change map for % of Bachelors Degrees
MapBachelors <- EduMap(varsJoined)
MapBachelors
ExportMaps(MapBachelors, "BachelorsMap.svg")

# Displays and Exports the change map for % of Non-White Residents
MapNonWhite <- RaceMap(varsJoined)
MapNonWhite
ExportMaps(MapNonWhite, "NonWhiteMap.svg")

# Displays and Exports the change map for Median Age
MapMedAge <- AgeMap(varsJoined)
MapMedAge
ExportMaps(MapMedAge, "AgeMap.svg")

# Displays and Exports the change map for Per Capita Income
MapPerCapIncome <- IncomeMap(varsJoined)
MapPerCapIncome
ExportMaps(MapPerCapIncome, "IncomeMap.svg")

# Displays and Exports the change map for Median Household Income
MapMedHouseIncome <- MhiMap(varsJoined)
MapMedHouseIncome
ExportMaps(MapMedHouseIncome, "MHIMap.svg")

# Displays and Exports the change map for Median Home Value
MapMedValue <- ValueMap(varsJoined)
MapMedValue
ExportMaps(MapMedValue, "ValueMap.svg")

# Exports data used for the final two maps
ModifiedData <- GentData(varsJoined)
st_write(ModifiedData, "FinalData.geoJson")

# Displays and Exports a synthesized Gentrification map based on values of the previous maps
MapGentri <- GentMap(GentData(varsJoined))
MapGentri
ExportMaps(MapGentri, "FinalGentMap.svg")

# Not Functional
# Displays and Exports a raster-interpolated Gentrification map based on centroids
MapInterp <- InterpMap(GentData(varsJoined))
MapInterp
ExportMaps(MapInterp, "InterpMap.svg")
```

