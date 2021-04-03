# Segmentation and clustering of Darmstadt area - Data

Our approach is to create a grid of segments of the Darmstadt area. 
We will use the Foursquare location data to assign to each segment 
the venues which are close to the segment center and their associated categories.
We will then cluster the segments using the weight of the nearby venue categories, 
i.e. the information of how much a certain category
is represented in a given segment.

In the end, a map with the segment centers will be provided. 
The segment centers will be colored depending on the cluster they belong to.
For every cluster a bar plot will be provided which will show the 
distribution of venue categories of the particular cluster.
