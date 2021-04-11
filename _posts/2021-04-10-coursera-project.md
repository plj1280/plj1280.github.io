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

You can already make out some interesting geographic patterns in the data. There's a large cluster of non-white population in the southeast, and smaller clusters in the center. There's some clusters of high-percentage white population in the north and southwest, and the rest fall into the range between 20%-70%. Interestingly, income seems to vary on smaller scales than education or ethnicity. Education and income seem correlated, and both are maybe weakly correlated with ethnicity. I'll return to these correlations later in the analysis

#### Foursquare Venue Data
I wanted to predict demand for Chipotle, but it turns out that _supply_ is much easier to measure. I could simply count up the number of restaurants in a given area. Then, we just proceed with the assumption that the actual supply roughly indicates the demand. To predict demand specifically for Chipotle, it would probably be best to use the supply of Chipotle. However, existing Chipotle locations in Queens are just too sparse to be a useful measure throughout the borough. Instead, I decided to consider the supply of "Chipotle-like" restaurants.

Originally, I was going to manually select known competitors of Chipotle to serve as the measure of demand. However, this proved tedious and the results were still too sparse. Instead, I decided to include restaurants based on similarities in customer-base by leveraging the Foursquare data. The method was as follows:
1. Select all Chipotles in Queens
2. For each Chipotle, find all the user-created lists they are a part of
3. For all lists, count up the venue categories that occur
4. For each tract, count the number of venues within a certain radius that belong to one of the most frequently occurring categories
The top 10 most frequently occurring venue categories are shown below:
<table border="1" class="dataframe">\n  <thead>\n    <tr style="text-align: right;">\n      <th>Categories</th>\n      <th>Count</th>\n    </tr>\n  </thead>\n  <tbody>\n    <tr>\n      <td>Mexican Restaurant</td>\n      <td>706</td>\n    </tr>\n    <tr>\n      <td>Asian Restaurant</td>\n      <td>319</td>\n    </tr>\n    <tr>\n      <td>Bar</td>\n      <td>122</td>\n    </tr>\n    <tr>\n      <td>Pizza Place</td>\n      <td>100</td>\n    </tr>\n    <tr>\n      <td>American Restaurant</td>\n      <td>96</td>\n    </tr>\n    <tr>\n      <td>Italian Restaurant</td>\n      <td>94</td>\n    </tr>\n    <tr>\n      <td>Food &amp; Drink Shop</td>\n      <td>92</td>\n    </tr>\n    <tr>\n      <td>Vegetarian / Vegan Restaurant</td>\n      <td>81</td>\n    </tr>\n    <tr>\n      <td>Seafood Restaurant</td>\n      <td>79</td>\n    </tr>\n    <tr>\n      <td>Dessert Shop</td>\n      <td>73</td>\n    </tr>\n  </tbody>\n</table>
