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
{% include table1.html %}

Then, I used the top 5 categories to get the restaurant supply density, shown in the following map:
{% include fig2.html %}

## Analysis
#### Correlation Analysis
As we saw in the maps previously, there appeared to be some correlations among the features. First, I simply plotted the correlation matrix as a heatmap to get an overview of the relationships.
![figm1](/images/figm1.png)

The largest correlations are education with income, and education with ethnicity, both at 0.5.
Income also shows a large negative correlation (-0.39) with population density.
Venue density shows small to moderate correlations with all features except median age.

Since there were some moderate correlations between our features, I calculated the principal components of the standardized feature set to see if there were redundancies I should remove to reduce the dimensionality.
The smallest component explained 7% of the total variance, so I decided that the variance was well-enough spread across the features to proceed without projection onto the principal components.

#### Cluster Analysis
Now, I'll finally get to one of the methods I used to try to answer the original question. Basically, I wanted to find locations where I expect there to be relatively high demand for Chipotle, and relatively low supply of competing restaurants. The first method I used was to cluster tracts by similarity in demographics, and then look for tracts with large, negative deviations from their cluster mean. The idea is, tracts in the same cluster should have similar demands due to demographic similarity, so a lower supply than expected might mean a good opportunity.

I chose to go with a simple k-means clusterings with 4 clusters, based on diminishing returns for higher cluster numbers. The results are shown in the following map and table:

{% include fig4.html %}
{% include table1.html %}

Next, the negative cluster deviations:

{% include fig5.html %}

The largest deviation occurs in the center of the map, next to the southeast end of Flushing Meadows Park.
A band of deviation extends east from this point. There's a similar, but less intense band running along the northern border of the borough.
The rest of the large deviation tracts are more scattered, with the most notable occurring in the western and northwestern sections.

#### Linear Regression
Next, I created a linear regression model to predict venue density from the demographic data.
I ran a grid search with 10-fold cross-validation over regularization parameter and polynomial degree.
As shown in the following figure none of the models fit the data particularly well, with a max mean $R^2$ of 0.17.
The best combination of high mean $R^2$ and relatively low standard deviation was the model with polynomial degree 2 and regularization parameter 100.
I trained that model on the entire dataset and then picked out the census tracts with large negative residuals
[figm3](/images/figm3.png)

{% include fig6.html %}

The residuals show a similar pattern as the cluster deviations, with notable bands running east from the center.
For the residuals the cutoff is sharper and fewer scattered tracts appear.

## Results and Conclusion
The final result is the following figure, which shows the existing Chipotle locations overlayed on maps of our two measures of unmet demand. Stakeholders could use this map to identify promising areas where there is high demand, low competition, and no existing locations. On first inspection, the band running east from Flushing Meadows Park, and the band to the north of that, seem like excellent candidates since they show up in both models of unmet demand and there are no Chipotle locations in the northeast quadrant of the map.

Obviously making a decision on opening a new location requires more information than was considered here, including company financials, real estate price and availability, and proximity to transportation. Nevertheless, the results presented here could be useful for identifying some areas to do more detailed research and analysis on.

{% include figure7.html %}

This concludes my post about my data science project. Thanks for reading!

If you have any questions about the APIs used or the data analysis and visualizations, check out the [Jupyter notebook](https://github.com). Otherwise, feel free to send comments or questions to me at plj1280 at gmail.
