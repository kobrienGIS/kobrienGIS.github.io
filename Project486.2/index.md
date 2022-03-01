# Baltimore City Relationship between Income and 311 Service Calls (Lab 4 R Markdown)

**Project description:** I chose to investigate the relationship between income levels and 311 service calls in Baltimore City Census Tracts in 2019. The following script uses get_acs() to retrieve ACS survey data and reads in a shapefile of counts by polygon for 311 calls from the Baltimore City Government's data portal. ggplot2, ggiraph, and patchwork are used to visualize the maps to hopefully show some sort of visual relationship between the datasets.

### Project Code

```r
# Load packages
library(tidycensus)
library(tidyverse)
library(dplyr)
library(sf)
library(ggplot2)
library(crsuggest)
library(ggiraph)
library(patchwork)
library(tigris)
options(tigris_use_cache = TRUE)

# Obtain Baltimore City Census Tract income data
BC_income <- get_acs(
  geography = "tract", 
  variables = "B19013_001",
  state = "MD",
  county = "Baltimore City",
  year = 2019,
  geometry = TRUE,
  resolution = "20m"
)

# Determine best CRS for the data. NAD83(HARN) / Maryland.
suggest_crs(BC_income)

# Visualize the income data using ggplot() and geom_sf()
IncomeMap <- BC_income %>%
  ggplot(aes(fill = estimate)) + 
  geom_sf(color = NA) + 
  # NAD83(HARN) / Maryland
  coord_sf(crs = 2804) + 
  geom_sf_interactive(aes(data_id = estimate), size = 0.1) +
  scale_fill_viridis_c(option = "inferno") + 
  theme_void() + 
  labs(fill = "Income\nEstimates")

# Load Baltimore City Census Tracts shapefile with 311 call point counts
Callsshape <- st_read("/Users/kyleobrien/Downloads/Lab4RCode/BaltimoreCityCounts/BCCenusTractCounts.shp")

# Visualize the 311 call count data again using ggplot() and geom_sf()
CallsMap <- Callsshape %>%
  ggplot(aes(fill = NUMPOINTS)) + 
  geom_sf(color = NA) + 
  # NAD83(HARN) / Maryland
  coord_sf(crs = 2804) + 
  geom_sf_interactive(aes(data_id = NUMPOINTS), size = 0.1) + 
  scale_fill_viridis_c(option = "inferno") + 
  theme_void() + 
  labs(fill = "311\nCalls")

# Call both maps with girafe() and include interactive highlighting
girafe(ggobj = IncomeMap + CallsMap, width_svg = 10, height_svg = 5) %>%
  girafe_options(opts_hover(css = "fill:cyan;"))

```
Unfortunately, the maps do not appear to suggest any obvious visual relationship between the variables. More data manipulations should be carried out to determine the degree of their statistical relationship with each other. 

## Write-up

- What data did you use and where did it come from?

I used income estimate data from the 2019 ACS for Baltimore City by Census Tract along with 311 calls data from the Baltimore City Government data portal. I transformed the tabled CSV data into points in QGIS and then used Count Points in Polygons with a Baltimore City Cenus Tracts GEOJSON file downloaded from the Maryland Government to create a 311 Call Counts by Census Tract layer. I then imported this shapefile back into RStudio to visualize it similarly to how I visualized the ACS data.

- Why did you decide to test this relationship?

I decided to test this relationship because I am frequently interested in the effects of various levels of income and how that determines many other aspects of life. I found the 311 calls data to be interesting because it reflects a variety of non-emergency calls for maintenance, cleanup, and other service requests, giving some insight into how well kept and how well invested an area is. 

- Why did you expect there to be a relationship?

I expected there to be a relationship between these two datasets because I inferred that income levels in a region would likely have some sort of effect on the quality of public services and utilities in the surrounding area. In areas with high income levels, I would expect to see fewer calls for service as public spaces are constructed better and cleaned frequently. The inverse would be true, with low income areas seeing much more service calls due to the poorer quality of infrastructure and waste management. 

- How did you test for a relationship? This one is really important.

For the time being, I was unable to implement any statistical methods for testing for a relationship due to time constraints and lack of know-how. If I were able to, I would include some form of regression analysis to see how closely related these variables are and test for the significance level of the relationship. In lieu of this, I decided to use ggiraph to offer a basic visual comparison of the two datasets and tried to interactively link the two to display their values when highlighted. This offers surface-level means for comparison and identification of some correlation, but it will not yet be statistically valid.



