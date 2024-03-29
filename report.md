# Enhanced location information for potential home buyers (application on german city using machine learning)

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
(see image below) of the area. 
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

<figure>
  <img src=./plots/grid_darmstadt_rect.JPG alt="my alt text"/>
  <figcaption> Fig. 1 - Grid of segments covering Darmstadt area</figcaption>
</figure>
<br/><br/>

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

<figure>
  <img src=./plots/top_cats.jpg alt="my alt text"/>
  <figcaption> Fig. 2 - Venue categories with the highest frequency occurence</figcaption>
</figure>
<br/><br/>

Ideally, we need a few categories and they should have a broader sense. For example, it does not make
a difference for the purpose of this study, if a venue is an "Italian restaurant" or a "French restaurant" or a "Café",
as long as it is a place where one can consume a beverage or a dish.

Thus,  a grouping of the venues is performed, as shown in the following table, leading to eight venues in total.

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

Let's start inspecting the results by examining some representative cases.
 
## Operator="sum", k=4

In all four clusters, the category "food service" is represented with the heighest weight. 
The dominant cluster is cluster 2, which is different from the other cluster in that the weights of
the other categories (not "food service") are higher. In this case, the over-representation of the
category "food service" in our data is clear. We also observe clearly the difference 
between the city centers, where "food service" is very dominant and peaked, 
and the surrounding areas, where the category distribution inside the clusters
is not as peaked. 

<figure>
  <img src=./plots/sum_4_map.jpg alt="my alt text"/>
  <figcaption> Fig. 3 - Map using k=4 and operator="sum"</figcaption>
</figure>

<figure>
  <img src=./plots/sum_cluster_4.png alt="my alt text" width="700"/>
  <figcaption>Fig. 4 - k=4, operator="sum": category distribution in clusters.</figcaption>
</figure>

## Operator="mean", k=4

The dominant cluster is cluster 3, which is very different from the other clusters, in that
the category distribution is more uniform and not as peaked as in the other clusters. 
We observe that the clusters 0, 1 and 2 
contain almost purely "food service", "sports" and "outdoor".
In contrast to the case with operator "sum" and k=4, 
the clustering here leads to more differentiation at the city borders and in the areas beyond the cities, whereas
the cluster distribution inside the city and residential areas is quite uniform and dominated by cluster 3.

<figure>
  <img src=./plots/mean_4_map.JPG alt="my alt text"/>
  <figcaption> Fig. 5 - Map using k=4 and operator="mean"</figcaption>
</figure>

<figure>
  <img src=./plots/mean_cluster_4.png alt="my alt text" width="700"/>
  <figcaption>Fig. 6 - k=4, operator="mean": category distribution in clusters.</figcaption>
</figure>

## Operator="mean", k=8

If we look at the category distribution inside clusters, we see that seven out of eight clusters contain only one dominant category.
Cluster 3 is slightly different and contains two dominant categories, "food service" and "basics", 
which are represented with approximately the same weight.
Based on my knowledge of the area, this cluster visualization makes sense and is a quite realistic representation.
For example, it is observed that cluster 2 with the dominant category "outdoor" is found in the areas outside cities.

<figure>
  <img src=./plots/mean_8_map.JPG alt="my alt text"/>
  <figcaption> Fig. 7 - Map using k=8 and operator="mean"</figcaption>
</figure>

<figure>
  <img src=./plots/mean_cluster_8.png alt="my alt text" width="700"/>
  <figcaption>Fig. 8 - k=8, operator="mean": category distribution in clusters.</figcaption>
</figure>

## Clustering performance

Plotting the silhouette score as a function of k for the operator "sum" (Fig. 7) shows that
the clustering quality decreases as k is increased.

<figure>
  <img src=./plots/inertias_silcoefs_sum.png alt="my alt text" width="300"/>
  <figcaption> Fig. 9 - operator="sum": inertia and silhouette score as a function of k</figcaption>
</figure>
<br/><br/>

When the silhouette score is plotted for the operator "mean" (Fig. 8), it is observed 
that the silhouette score increases a function of k and stabilizes for k values above 9.

<figure>
  <img src=./plots/inertias_silcoefs_mean.png alt="my alt text" width="300"/>
  <figcaption> Fig. 10 - operator="mean": inertia and silhouette score as a function of k</figcaption>
</figure>

# Discussion

## Effect of the aggregation operation

The aggregation operation leads to different clustering results. This is observed clearly by comparing 
the silhouette score as a function of k in Fig. 9 and in Fig. 10. 
It is also observed on the distribution of
the category in the clusters by comparing Fig. 4 with Fig 6.
Indeed, the use of the "sum" operator for this data leads to clusters which are less cohesive and less well separated 
then the clusters obtained with the use of the "mean" operator.
Furthermore, the results for the "median" operator are similar to the results
for the "mean" operator. Because of their similarity, these results are not
presented in detail in this report.


## Effect of the number of clusters

When the number of clusters is increased and when the mean (or median) operator is
used, the cluster cohesion as measured by the silhouette score improves, as shown in Fig. 10. This
is also observed in the distribution of venue categories in the clusters, by comparing Fig. 6 with Fig. 8. 
For k>8, the added value by increasing k further, is not that clear and the
cluster cohesivity as measured with the sihouette score does improve
substantially, as showin in Fig. 10.


# Conclusion

A segmentation and clustering analysis on the Darmstadt area using the Foursquare data is performed
with the aim of characterizing areas to help potential house buyers to determine good locations for 
housing purposes.
A spatial grid on the Darmstadt area is constructed and the data are properly mapped onto the grid using the haversine formulas.
The data are processed and organized in representative venue categories.
Using an aggregation operator, the venue categories are mapped onto the grid segments.
Finally, unsupervised clustering using the k-means method is applied on the data and the cluster results are analyzed
as a function of k and the aggregation operator.
It is found that a good cluster cohesion and realistic representation of the
area is obtained when the aggregation operator using averaging ("mean") and k=8. 

