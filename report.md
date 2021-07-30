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

![](https://github.com/theofn/coursera_public/blob/main/plots/grid_darmstadt_rect.JPG)

The advantage of the approach decribed previously is that it is very general and it can be applied
to an area which I know very well. Thus, I can verify that the results
are consistent with my knowledge of the area. Unfortunately, I could find
no other data sources, which could be useful for this study.


# Methodology

## Collection of venue data

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
to associate the venues found with Foursquare venue search to the correct segment. For every grid
node, the venue search returned a number of venues. The venues which are outside
the circular area defined by the 400 m radius where discarded. The other venues
are added as a row to a pandas dataframe. After iterating over all the grid nodes,
all the data are collected. However, some venues are collected more than once.
The duplicated are then removed from the data frame, so that every venue is unique.

## Grouping of venue categories

Using the method described above, a total number of 4216 venues are collected, associated 
with 820 segments. As expected, no venues are found in the vicinity of almost half of the nodes.
The nodes are located in forest areas or in agricultural areas in which no data Fousquare data exist.

Every venue is associated to a certain category. In our case, the venue data contain 471 categories,
which is probably a too high number for the clustering. Using all these categories does not produce
useful results. Furthermore, the category distribution is such that the most frequently found category by far is 
"Office" and exceeds by more than twice the second more frequent category, which is "Doctor's office", as it
can be seen in the figure below.

![](https://github.com/theofn/coursera_public/blob/main/plots/top_cats.jpg)

Ideally, we need no more than 10 categories and they should have a broader sense. For example, it does not make
a difference for the purpose of this study, if a venue is an "Italian restaurant" or a "French restaurant" or a "Café",
as long as it is a place where one can consume a beverage or a dish.

Thus,  a grouping of the venues is performed, as shown in the following table.

| New category group | Foursquare categories (or strings therein) | # entries |
| ----------- | ----------- | ----------- |
| Education/Childcare      | high/middle/elementary school, college, university, nursery, daycare, etc | 89 |
| Shopping   | mall, clothing, sporting goods, flower, tailor, gift shop, jewelry, shoe store, etc   | 160 |
| Basics      | supermarket, grocery, vegetable, butcher, drugstore, bakery, etc | 229 |
| Health      | hospital, doctor, pharmacy, dentist, medical | 254 |
| Food service | restaurant, café, cafeteria, burger, ice cream shop, pizza, coffee shop, coffee shop, etc | 707 |
| Entertainment | entertainment, jazz, concert, theater, pub, nightlife, nightclub, lound, etc | 224 |
| Sports | football, tennis, baseball, basketball, fitness, swimming, pool, stadium, etc | 135 |
| Outdoor | scenic, nature, national park, outdoors, river, zoo, castle, lake, etc | 126 |

As shown in the previous table, the category "food service" are over-represented in our data. 
This has implications for the analysis and needs to be taken into account when the clustering is done
and the results are analysed.

## Grouping by segment and evaluation of category weights

The next step to group the venue data by segment and calculate for every segment how much is the 
weight of every category. The calculation is an aggregation operation 
and transforms a collection of numbers to a single number.
There are several possibilities regarding aggregation. The following operators
are considered: sum, mean and median.

Using the sum operation results in representing the weight of each category using the absolute count of
venues. In this case, the category "Food service" is practically in all segments
the most abundant. Thus, all clusters are expected to have a high weight of "Food service" 
and smaller amounts of the other categories.
Depending on the clustering method, the category "Food service" could blur the effect of
the other categories.

Using the mean or median operation involves a normalization of the category count, as it
takes into account the total number of venue categories in every segment. It also makes sense
to use such an operator because it is expected that it is not possible to have a similar amount
of venues in all categories. However, the problem here is that in many segments the statistics
are poor: for example, there are segments with one outdoor venue. This can also lead to 
a clustering which is not very representative of the venue category distribution.

## Clustering

After the grouping-by-segment operation, the clustering of the venue data 
takes place using the k-means algorithm. In particular, the implementation of 
python package scikit-learn is used. To evaluate the clustering performance, and since the clustering is unsupervised,
the inertia (within-cluster sum-of-squares) and the silhouette coeficient are used.
In addition, for every cluster, the weight of each venue is evaluated and these
data are visualized using bar plots.
Finally, a map of the Darmstadt area with segments colored depending on
their cluster assignment is produced to visualize the spatial distribution.

# Results

## Effect of the aggregation operation

The aggregation operation leads to different clusterings. This is observed in one of the clustering performance metrics, which we consider: the silhouette score (aka silhouette coefficient). It is also observed on the distribution of the category in the clusters. For example, when the sum operator is considered, the silhouette score tends to decrease as a function of the number of clusters, while the inertia decreases.

![](https://github.com/theofn/coursera_public/blob/main/plots/inertias_silcoefs_sum.png)

This implies that the clusters become less cohesive and less well separated as the k, the number of clusters, increases.
On the other hand, when the mean operator is considered, the silhouette score increases as a function of the number of clusters and seems to reach a plateau when k is approximately 8.

![](https://github.com/theofn/coursera_public/blob/main/plots/inertias_silcoefs_mean.png)


## Effect of the number of clusters

If the number of clusters is increased, the performance metrics of the clustering improve. 




# Discussion

# Conclusion
