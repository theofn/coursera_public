# Segmentation and clustering of the Darmstadt area (Germany)

## Background

Buying a house or an appartment to live in is of paramount importance to most people. 
It is often the biggest investment in a lifetime and it determines one's living standards 
for decades. One of the important parameters which affects the property value and 
the quality of living is the location and the nearby surroundings.
In particular, the location determines if schools, public transport, shops, restaurants 
and other facilities or venues are in the surrounding area and at which distance.
For example, an ideal location for some people would be close to nature and having
to use often the car and commute would not be an issue. For others, an ideal location would be
close to shops or schools and they might like to minimise the use of car as possible.
So, it would be helpful to make such information easily available 
so that potential house or appartment buyers can make the right decision regarding
the property location.

## Problem

The aim of this project is to characterize areas on a map using the Foursquare data,
according to their vicinity to various venue categories, such as, childcare, sports,
shops, entairtainment, healthcare, etc. The areas should be characterized by an effective
radius, such that one can reach the venues within a reasonable amount of time, e.g.
no more than 15-20 minutes on foot. To this end, a grid approach will be used and will be
applied in particular for the Darmstadt area in Germany (which is well-known to the
author of this document). 

## Target audience

The target audience is potential buyers for housing reasons. 
Users of the results of this project will be able to identify areas which are a close match
to their needs or expectations.

# Data

My approach is to create a grid of segments 
(see image below) of the Darmstadt area. 
We will use the Foursquare location data API to assign to each segment 
the venues which are close to the segment center and their associated categories.
We will then cluster the segments using the weight of the nearby venue categories, 
i.e. the information of how much a certain category
is represented in a given segment.

In the end, a map with the segment centers will be provided. 
The segment centers will be colored depending on the cluster they belong to.
For every cluster a bar plot will be provided which will show the 
distribution of venue categories of the particular cluster.

Consider an example of how this information can look like. For simplicity, we assume here
that we have only 2 clusters. Using the bar plots
we will be able to see that cluster 1 contains 50% catering, 40% shops and
10% entertainment venues. Cluster 2 contains 30% sports, 30% outdoor and 40%
catering venues. On the map we will be able to draw hundreds of segments
and will assign each segment one of two colors, corresponding to the two clusters.

![](https://github.com/theofn/coursera_public/blob/main/grid_darmstadt.jpg)

The advantage of the approach decribed previously is that it is very general and it can be applied
to an area which I know very well. Thus, I can verify that the results
are consistent with my knowledge of the area. Unfortunately, I could find
no other data sources, which could be useful for this study.


# Methodology

The analysis begins by creating a uniform rectangular grid of nodes which includes
the city of Darmstadt and the nearby surrouding areas. All the sides
of the grid are 17.6 km long. 

The next step is to determine the size of the segments associated to each node.
On one hand, a certain granularity is desired, so that we have enough segments to cluster.
On the other hand, the segments should contain a high enough number of venues, so that
the statistical aggregation operations which take place on every segmen, such as summing or averaging, make sense.
After a number of trials, it became clear that a good radius is 400 m around every node. 
This results into a 40x40 grid.

The data are then collected by iterating on the grid and using the venue search request of the
Foursquare places API. The challenge here was to control the spatial extent of the 
returned venues. The API request returns venues which are "near" the input location, however
it is not possible to control the radius in a useful way, since the parameter "radius" is only 
valid for searches using the parameters "categoryId" or "query". 

I found a solution to this problem by using the haversine formulas. Thus, it was possible 
to associate the venues found with Foursquare venue search to the correct segment.


# Results

# Discussion

# Conclusion