# Motor Vehicle Fatality Rates in the U.S. from 1980 to 2020 (Animated GIF)

<img src="Lab3ges383.svg?raw=true"/>

## Project Description:
One of the primary issues concerning analyses of U.S. motor vehicle and travel safety is effectively visualizing trends over time. To allow for stable and reliable longitudinal comparison between states, I created a series of maps that display motor vehicle accident fatality rates by state from 1980 to 2020. This study utilized accident report data from all 50 states and D.C. compiled by the NHTSA alongside population data by state kept by the US Census Bureau to calculate rates of total fatalities resulting from traffic accidents normalized by population. Accident fatalities was chosen as the metric because deaths are unlikely to go unreported and do not require additional definition or specification. The maps were created in RStudio with a variety of packages including tigris, sf, dplyr, and ggplot2. The program joins state geographies using tigris with motor vehicle fatality rate data to plot and export the maps with ggplot2. The program also takes advantage of function calls to replicate the maps with five separate CSV inputs for each of the five decades of study. The findings demonstrate that motor vehicle fatality rates have steadily declined across the U.S. from 1980 to 2010, however fatality rates have momentarily increased in 2020. Wyoming, Mississippi, New Mexico, and South Carolina appear to have consistently remained among the most dangerous, while New England, Illinois, Minnesota, Utah, Washington, and Hawaii appear to lead in traffic safety. 

## Project Code:
Packages
```{r}
library(tidycensus)
library(tidyverse)
library(tigris)
library(dplyr)
library(sf)
library(ggplot2)
library(viridis)
```

Core Functions
```{r}
MapForEachYear <- function(csv) {
  # Data sources: 
  # https://www.nhtsa.gov/file-downloads?p=nhtsa/downloads/FARS/ 
  # https://www.census.gov/data/tables/time-series/dec/popchange-data-text.html
  
  # Reads CSVs for every decade. Data was cleaned and organized manually in Excel before input.
  fatalities <- read_csv(csv)

  # Calls Tigris geometry for the United States and District of Columbia, excludes Puerto Rico.
  us_states <- states(cb = TRUE, resolution = "20m") %>%
      filter(NAME != "Puerto Rico") %>%
      shift_geometry()
  
  # Joins the state geometries and CSV data.
  us_states_joined <- us_states %>%
      left_join(fatalities, by = "NAME")
  
  # Returns the map data for visualization.
  return(us_states_joined)
}

PlotMap <- function(map, year) {
  # Data attribution for use in the caption.
  nhtsa <- paste("Data Sources:", year, "FARS Reports, National Highway Traffic Safety Administration", sep =" " )
  census <- paste("&", year, "Decennial Survey, United States Cenus Bureau", sep =" ")
  
  # Visualizes the maps with a viridis color palette and gives them a title, legend, and caption.
  final_product <- ggplot(map) + 
      geom_sf(aes(fill = FATRATE), color = alpha("white", 1 / 2), size = 0.25) + 
      scale_fill_viridis(option = "inferno", direction = -1, limits = c(0, 60), breaks = c(10, 20, 30, 40, 50)) +
      theme_void() + 
      theme(text = element_text(size = 12, family = "serif")) +
      theme(plot.margin = unit(c(1, 1, 1, 1), "cm")) +
      labs(fill = "Deaths per\n100,000 Residents",
          title = paste("Motor Vehicle Fatality Rates in the US,", year, sep =" "),
          caption = paste(nhtsa, census, sep = "\n"))
  
  # Returns the final map product for export.
  return(final_product)
}

ExportMap <- function(mapproduct, filename) {
   # Exports each map frame to stitch together in an animated gif. The default path is set to the working directory.
   ggsave(
      filename,
      plot = mapproduct,
      scale = 1,
      dpi = 300,
      bg = "white"
   )
}
```

Main Code
```{r}
# Displays and Exports the map for 1980
Map1980 <- PlotMap(MapForEachYear("/Users/kyleobrien/Downloads/Project1/FARScsvs/FARS1980.csv"), "1980")
Map1980
ExportMap(Map1980, "Map1980.png")

# Displays and Exports the map for 1990
Map1990 <- PlotMap(MapForEachYear("/Users/kyleobrien/Downloads/Project1/FARScsvs/FARS1990.csv"), "1990")
Map1990
ExportMap(Map1990, "Map1990.png")

# Displays and Exports the map for 2000
Map2000 <- PlotMap(MapForEachYear("/Users/kyleobrien/Downloads/Project1/FARScsvs/FARS2000.csv"), "2000")
Map2000
ExportMap(Map2000, "Map2000.png")

# Displays and Exports the map for 2010
Map2010 <- PlotMap(MapForEachYear("/Users/kyleobrien/Downloads/Project1/FARScsvs/FARS2010.csv"), "2010")
Map2010
ExportMap(Map2010, "Map2010.png")

# Displays and Exports the map for 2020
Map2020 <- PlotMap(MapForEachYear("/Users/kyleobrien/Downloads/Project1/FARScsvs/FARS2020.csv"), "2020")
Map2020
ExportMap(Map2020, "Map2020.png")
```
