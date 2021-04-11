---
title: Determining the Optimal Location for a new Chipotle Restaurant in Queens, NY
layout: default
---
{% include cdn.html%}
# {{page.title}}
## Introduction
For my capstone project for the IBM Data Science Certification, I used demographic data from the US Census Bureau and venue data from Foursquare to try to determine attractive candidate locations for a new Chipotle restaurant.
In this post, I'll discuss the data sources I used and the analysis I did. But mainly, I'll show off some cool interactive maps I made.
For more details, check out this [Jupyter notebook](https://github.com/plj1280) that contains all the steps of data processing, analysis, and visualization, and this [formal report](https://github.com/plj1280) version of the results.

## Data
#### Census Tract Demographics
I obtained demographic data for all census tracts in Queens, NY using the US Census Bureau API for [American Community Survey 5-year Data](https://www.census.gov/data/developers/data-sets/acs-5year.html). This dataset is pretty cool because census tracts are quite fine-grained geographical units.
However, because of the small sample sizes involved, tract-level data is only published with 5 years of accumulated surveys. In any case, I used the 5-year data published in 2019 to get the following for each census tract:
- median age
- median income
- percent of population identifying as white
- percent of population with bachelor's degree or higher
- population density

The idea is to use these variables to predict demand for Chipotle. But first, I needed to get the geometry data for the census tracts to make maps.

#### Census Tract Geometry
Conveniently, the US Census Bureau also has a [REST API service](https://www.census.gov/data/developers/data-sets/TIGERweb-map-service.html) to return geospatial and geometry data for all the geographical divisions the census uses. The API can return the data in the geoJSON format, so I was able to plug it directly into Plotly to draw choropleth maps of the census tracts. So, without further adieu, here are some maps showing census data for census tracts in Queens, NY:

{% include fig1.html %}

  
