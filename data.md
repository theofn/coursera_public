# Segmentation and clustering of Darmstadt area - Data

Our approach is to create a grid of segments 
(you can think of it as a kind of neighborhood) of the Darmstadt area. 
We will use the Foursquare location data to assign to each segment 
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
