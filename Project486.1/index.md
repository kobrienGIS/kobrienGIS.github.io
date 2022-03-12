# Income to Poverty Level Ratio in Harford County, Maryland 

<img style="border:3px solid black;" src="Screen Shot 2022-02-21 at 10.09.36 PM.png?raw=true"/>

## Project description: 
This map was an attempt to create an interactive map using RStudio's Rmarkdown file type for organization and was completed with the help of an R tutorial text created by Kyle Walker. The script uses tidycensus to pull census data for Harford County, Maryland that conveys the ratio of income to poverty levels in the past 12 months per census tract. The map was then interactively visualized using tools contained in leaflet. 

## Project Script:
```r
# Load necessary R packages
library(leaflet)
library(tidycensus)

# Generates a list table of possible variables to use in get_acs()
tidyvars <- load_variables(2019, "acs5", cache = TRUE)
View(tidyvars)

# Loads user census api key
#census_api_key("INSERT API", install = TRUE, overwrite = TRUE)

# Retrieves data for Harford County for ratio of income to poverty level in the past 12 months
harford_rincpov <- get_acs(
  geography = "tract",
  variables = "C17002_001",
  year = 2019,
  state = "MD",
  county = "Harford",
  geometry = TRUE
)
View(harford_rincpov)

# Introduces the magma color palette used in the chapter
pal <- colorNumeric(
  palette = "magma",
  domain = harford_rincpov$estimate
)

# Visualizes the map with leaflet using the color palette and Harford County data
leaflet() %>%
  addProviderTiles(providers$Stamen.TonerLite) %>%
  addPolygons(data = harford_rincpov,
              color = ~pal(estimate),
              weight = 0.5,
              smoothFactor = 0.2,
              fillOpacity = 0.5,
              label = ~estimate) %>%
  # Creates a legend to accompany the map data
  addLegend(
    position = "bottomright",
    pal = pal,
    values = harford_rincpov$estimate,
    title = "Ratio of Income<br/>to Poverty Level<br/>in the last 12 Months"
  )
```
