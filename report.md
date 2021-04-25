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

# Segmentation and clustering of Darmstadt area - Data

Our approach is to create a grid of segments 
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

# Methodology

# Results

# Discussion

# Conclusion
