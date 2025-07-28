# Lede Project 3: How densely populated are flood hazard areas in Germany?

This is my submission of my third project during the Lede Program for Data Journalism (https://ledeprogram.com/). 

Text and data have not been editorially reviewed or fact-checked.

The data generated from QGIS can be found here: https://github.com/sophiabmnn/PopulationDensityFloodHazardAreas-Lede-Project-3/tree/main/PopulationDensityValuesPerState 

You can find my notebook here: https://github.com/sophiabmnn/PopulationDensityFloodHazardAreas-Lede-Project-3/blob/main/Project3_PopulationDensity_Calculations.ipynb

The results of my analysis are stored here: https://github.com/sophiabmnn/PopulationDensityFloodHazardAreas-Lede-Project-3/tree/main/Results 

## Aim

Germany has been repeatedly hit by severe flood disasters in recent years. In July 2021, at least 135 people died in the Ahr Valley in western Germany. Climate change is making such extreme weather events increasingly likely. My research question therefore is: How densely populated are flood hazard areas in Germany?

## Proceeding

### Data sources

- Data Source for Country/State Borders: For mapping the borders of German states, I used the „Admin 1 - States, Provinces“-File by Natural Earth (https://www.naturalearthdata.com/downloads/10m-cultural-vectors/10m-admin-1-states-provinces/).
- Data Source for Flood Hazard Areas: The „Bundesanstalt für Gewässerkunde“, an official authority, offers different GML-files for Flood Hazard Areas: Flood Hazard Area Fluvial and Flood Hazard Area Sea, each with different risk levels (https://geoportal.bafg.de/inspire/download/NZ/hazardArea_flood/datasetfeed.xml). My analysis is based on the Flood Hazard Area Fluvial - Scenario High. Therefore, the results restricted to Fluvial Floods. „Scenario High“ means in that areas every 10 to 30 years a flood is expected (https://geoportal.bafg.de/arcgis/rest/services/INSPIRE/NZ/MapServer/1).
- Data Source for Population Density: For an estimation of the population density, I used the „Global Human Settlement Layer“ (GHSL) from Copernicus (https://human-settlement.emergency.copernicus.eu/download.php?ds=pop). I used the GHS-Pop data for 2025, with a resolution of 100m and Mollweide projection.

### Preparatory Steps

Country/State Borders-Data:

I did not only want to show the overall population density in Germany, but also regional differences. Therefore, I had isolate each German state from the „Admin 1 - State, Provinces“-File: I opened the overall file in QGIS, filtered for each state and saved the results as shapefiles. I also isolated Germany as a whole county and saved it as a shapefile.

Flood Hazard Area-Data:

- First, I converted the Flood Hazard Area-Data with the help of the Geopandas-Library into a GEOJSON-file. You can find my notebook here: https://github.com/sophiabmnn/PopulationDensityFloodHazardAreas-Lede-Project-3/blob/main/Project3_Convertion-GML-GEOJSON.ipynb 
- Then, I uploaded the GEOJSON-File into Mapshaper. I reduced the number of boundaries in the data with the command „dissolve2“ and simplified the dataset afterwards (https://mapshaper.org/).
- As a next step, I uploaded the simplified file into QGIS and checked its validity. There was indeed a problem with one tiny point, so I fixed the geometry of the file by using „fix geometry“ in the processing toolbox in QGIS.
- In a last step, I cut the file into each of the 16 German states by using the GDAL „Clip Vector by Mask Layer“-tool in QGIS. As a mask layer, I used the prepared shape files from the countries/state-border data. Restricton: A part of the border rivers were lost during that process, as they weren’t included in the county/state-borders-data.

Population Density-Data:

- The Population Density Data is provided in grids, with Germany being represented by four different grids. In a first step, I therefore uploaded these four grids into QGIS and merged them into one file. 
- In a second step, I filtered for the German population density data by using the processing tool „clip raster by mask layer“ in QGIS. My mask layer was the Germany-shapefile from the Country/State Borders-Data.
- For the calculations, it was necessary to classify the population density data: So far, the file just showed the estimated number of people living in each cell (see documentation here: https://human-settlement.emergency.copernicus.eu/ghs_pop2023.php).
- For the classification, I leaned on the definition from the European Union: Rural grids are thereby defined as having less than 300 inhabitants per square kilometer, urban clusters have more than 300 and urban centers more than 1500 (https://ec.europa.eu/eurostat/statistics-explained/index.php?title=Territorial_typologies_manual_-_cluster_types&oldid=649958).
- As I downloaded the 100m-resolution dataset, each cell represented 100×100 = 10,000 square meters. That means for data: Less than 3 inhabitants are equivalent to a rural area, between 3 and 15 are classified as an urban cluster, more than 15 as an urban center.
- I classified the data in QGIS, using the Raster Calculator. I set all values equal to zero to 0, everything between 0 and 3 to 1, everything between 3 and 15 to 2, and everything above 15 to 3. Documentation: https://download.qgis.org/qgisdata/QGIS-Documentation-2.6/live/html/en/docs/user_manual/working_with_raster/raster_calculator.html  
- For the further calculations, I only needed the population density in the flood hazard areas of each state. By using the GDAL „Clip Raster by Mask Layer“-tool, I cut the data, using the flood hazard area of each state data as a mask. In the end, I had the population density in the flood hazard zones for each state.

### Calculations

- I used the QGIS-Function Raster Layer Unique Values-Report to calculate how many cells in each of the fluvial hazard zones in whole Germany as well as in the respective states are classified as rural, as urban cluster and urban center. I then exported these tables as CSV. You can find the results here: https://github.com/sophiabmnn/PopulationDensityFloodHazardAreas-Lede-Project-3/tree/main/PopulationDensityValuesPerState 
- With Python, I calculated how many percent of the flood hazard zones in each state can be classified as „not populated“, "rural area“, „urban cluster“ and „urban center“. You can find my notebook here: https://github.com/sophiabmnn/PopulationDensityFloodHazardAreas-Lede-Project-3/blob/main/Project3_PopulationDensity_Calculations.ipynb 

## Results

82.02 percent of the German high risk flood hazard areas are not populated. However, 11.51 percent have at least a low population density („rural area“), 4.59 percent can be defined as urban area, and 1.88 percent even as urban center. 

With regard to the states, Nord Rhine-Westfalia has the biggest area of populated high risk flood hazard areas – but it is also one of the biggest states. Looking only at the share of each category in the flood hazard areas themselves, Hamburg’s flood zones are most densely populated, with 45.79 percent being populated. On the other side, Saarland’s high risk flood hazard areas are not populated, according to this analysis.

## Learnings

The aim of this project was to become familiar with working with satellite (raster) data in QGIS. For me, the biggest lesson was that it is very difficult to scale up this type of analysis using the chosen methods. I had to do a lot of manual cutting and calculating. This, in turn, makes the analysis more difficult to reproduce. So if I had more time, I would try to repeat this analysis using methods outside of QGIS.