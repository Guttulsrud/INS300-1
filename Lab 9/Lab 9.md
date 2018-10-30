# Lab 8 INS300-1
# Agenda
* Usupervised Machine Learning
    * Dimensionality Reduction
        * Principal Component Analysis
    * Clustering
        * k-Means Clustering
        * Hierarchical Clustering

# Unsupervised Learning

For supervised learning, we have one or more targets we want to predict using a set of explanatory variables. But not all data analysis consists of making prediction models. Sometimes we are just trying to find out what structure is actually in the data we analyze. There can be several reasons for this. Sometimes unknown structures can tell us more about the data. Sometimes we want to explicitly avoid an unknown structure
(if we have datasets that are supposed to be similar, we don’t want to discover later that there are systematic differences). Whatever the reason, unsupervised learning concerns finding unknown structures in data.



## Dimensionality Reduction
Dimensionality reduction, as the name hints at, are methods used when you have high-dimensional data and want to map it down into fewer dimensions. The analysis usually just transform the data and doesn’t add anything to it. It possibly removes some information, but by reducing the number of dimensions it can be easier to analyze.

### Principal Component Analysis (PCA)
The idea behind pca is to `PCA`  is to map your data from one vector space to another of the same dimensionality as the first. So it doesn’t reduce the number of dimensions as such. However, it chooses the coordinate system of the new space such that the most information is in the first coordinate, the second most information in the second coordinate, and so on.

Because the `PCA` just transforms your data, your data has to be numerical vectors, to begin with. For categorical data, you will need to modify the data first. One approach is to represent factors as a binary vector for each level, as is done with model matrices in supervised learning. If you have a lot of factors in your data, though, PCA might not be the right tool.

```R
library(magrittr)
library(ggplot2)
library(dplyr)

library(mlbench)
```

### IRIS dataset

```R
iris %>% head
```
To see if there is information in the data that would enable us to distinguish between the three species based on the measurements, we could try to plot some of the measurements against each other.
```R
iris %>% 
  ggplot() + 
  geom_point(aes(x = Sepal.Length, y = Sepal.Width, colour = Species))

iris %>% 
  ggplot() + 
  geom_point(aes(x = Petal.Length, y = Petal.Width, colour = Species))
```

It does look as if we should be able to distinguish the species. Setosa stands out on both plots, but Versicolor and Virginia overlap on the first.

Since PCA only works on numerical data, we need to remove the Species parameter, but after that, we can do the transformation using the prcomp function:

```R
?prcomp
pca <- iris %>% select(-Species) %>% prcomp
```

The object that this produces contains different information about the result. The standard deviations tell us how much variance is in each component and the rotation what the linear transformation is.

![](https://i.imgur.com/S72sG4g.png)


The first thing you want to look at after making the transformation is how the variance is distributed along the components. If the first few components do not contain most of the variance, the transformation has done little for you.

```R
mapped_iris <- pca %>% predict(iris) 
mapped_iris %>% head
```


```R
mapped_iris %>%
    as.data.frame %>% 
    cbind(Species = iris$Species) %>% 
    ggplot() + 
    geom_point(aes(x = PC1, y = PC2, colour = Species))
```

![](https://i.imgur.com/luOz8eD.png)

```R
pca <- iris %>% select(-Species) %>% prcomp

mapped_iris <- pca %>% 
  predict(iris) %>%
  as.data.frame %>% 
  cbind(Species = iris$Species)

iris_rotation <- pca$rotation %>% as.data.frame()

ggplot() + 
  geom_point(aes(x = PC1, y = PC2, color = Species), data = mapped_iris) +
  geom_segment(aes(xend = PC1, yend = PC2), x = 0, y = 0, arrow = arrow(), data = iris_rotation) + 
  geom_text(aes(x = PC1, y = PC2),label = rownames(iris_rotation), data = iris_rotation)
```

![](https://i.imgur.com/afZvnNl.png)



### HouseVotes84 dataset from mlbench
This data set includes votes for each of the U.S. House of Representatives Congressmen on the 16 key votes identified by the CQA.

Let's load the HouseVotes84 dataset into R:

```R
data(HouseVotes84)
```

Now let us take a look at the data using the head, and/or View command:
```R
HouseVotes84 %>% head
HouseVotes84 %>% View
```

The data contains the votes cast for both republicans and democrats on 16 different proposals. The types of votes are yea, nay, and missing/unknown. Now, since votes are unlikely to be accidentally lost, missing data here means someone actively decided not to vote, so it isn’t really missing. There is probably some information in that as well.

Before we continue to process the data, what does the column mean? Well, using the `?HouseVotes84` will give us a hint.

```R
?HouseVotes84
```

So why don't we include the column names as headers for the dataset? We can do it like this:

```R
header <- c("Class",
            "handicapped-infants",
            "water-project-cost-sharing",
            "adoption-of-the-budget-resolution",
            "physician-fee-freeze",
            "el-salvador-aid",
            "religious-groups-in-schools",
            "anti-satellite-test-ban",
            "aid-to-nicaraguan-contras",
            "mx-missile",
            "immigration",
            "synfuels-corporation-cutback",
            "education-spending",
            "superfund-right-to-sue",
            "crime",
            "duty-free-exports",
            "export-administration-act-south-africa")

colnames(HouseVotes84) <- header
```

Let's view the dataset once more, to see the change of header name:

```R
View(HouseVotes84)
```

_Now back to processing the data:_

An interesting question we could ask is whether there are differences in voting patterns between republicans and democrats. We would expect that, but can we see it from the data?

The individual columns are binary (_well, trinary if we consider the missing data as actually informative_) and do not look very different between the two groups, so there is little information in each individual column. We can try doing a PCA on the data, just like we did before:

```R
HouseVotes84 %>% select(-Class) %>% prcomp

# Error in colMeans(x, na.rm = TRUE): 'x' must be numeric
```

Okay, R is complaining that the data isn’t numeric. We know that PCA needs numeric data, but we are giving it factors. We need to change that so we can try to map the votes into zeros and ones:

We can use the function `apply()`. This function is used to apply a function to a matrix and what it does depends on the dimensions we tell it to work on. It can summarize data along rows or along columns, but if we tell it to work on both dimensions, that is the c(1,2) argument to the function, it will apply the function to each element in the matrix. The transformation to do is just a function we give `apply()`.

```R
HouseVotes84 %>%
select(-Class) %>%
apply(c(1,2), . %>% { ifelse(as.character(.) == "n", 0, 1) }) %>% prcomp

# Error in svd(x, nu = 0): infinite or missing values in 'x'
```

That doesn’t work either, but now the problem is the missing data. We have mapped nay to 0 and yea to 1, but missing data remains missing.

We should always think carefully about how we deal with missing data, especially in a case like this where it might actually be informative. One approach we could take is to translate each column into three binary columns indicating if a vote was cast as yea, nay, or not cast.

I have left that as an exercise. Here I will just say that if someone abstained from voting, then they are equally likely to have voted yea or nay and translate missing data into 0.5.

```R
vote_patterns <- HouseVotes84 %>%
select(-Class) %>%
apply(c(1,2), . %>% { ifelse(as.character(.) == "n", 0, 1) }) %>% apply(c(1,2), . %>% { ifelse(is.na(.), 0.5, .) })

pca <- vote_patterns %>% prcomp
```

Now we can map the vote patterns onto the principal components and plot the first against the second:

```R
mapped_votes <- pca %>% predict(vote_patterns) 

mapped_votes %>% as.data.frame %>%
cbind(Class = HouseVotes84$Class) %>%
ggplot() +
geom_point(aes(x = PC1, y = PC2, colour = Class))
```
It looks like there is a clear separation in the voting patterns, at least on the first principal component. This is not something we could immediately see from the original data.

![](https://i.imgur.com/JSsVL78.png)

But which patterns matter with regards to voting Democrat or Republican? We can use the same method 

```R
votes_rotation <- pca$rotation %>% as.data.frame()

mapped_votes %>% as.data.frame %>%
  cbind(Class = HouseVotes84$Class) %>%
  ggplot() +
  geom_point(aes(x = PC1, y = PC2, colour = Class)) +
  geom_segment(aes(xend = PC1, yend = PC2), x = 0, y = 0, arrow = arrow(), data = votes_rotation) +
  geom_text(aes(x = PC1, y = PC2),label = rownames(votes_rotation), data = votes_rotation)
```

![](https://i.imgur.com/MYmIddF.png)


## Clustering
Clustering methods seek to find similarities between data points and group data according to these similarities. Such clusters can either have a hierarchical structure or not; when the structure is hierarchical, each data point will be associated with several clusters, ordered from the more specific to the more general, and when the structure is not hierarchical any data point is typically only assigned a single cluster. The next sections describe two of the most popular clustering algorithms, one of each kind of clustering.


### K-means Clustering
In k-means clustering you attempt to separate the data into k clusters, where you determine the number k. The data usually has to be in the form of numeric vectors. Strictly speaking, the method will work as long as you have a way of computing the mean of a set of data points and the distance between pairs of data points. The R function for k-means clustering, kmeans, wants numerical data.

![](https://i.imgur.com/qvKsumu.png)

The algorithm essentially works by first guessing at k “centers” of proposed clusters. Then each data point is assigned to the center it is closest to, creating a grouping of the data, and then all centers are moved to the mean position of their clusters. This is repeated until an equilibrium is reached.

![](https://i.imgur.com/LYhkvUk.gif)

Let’s see it in action. We use the iris dataset, and we remove the Species column to get a numerical matrix to give to the function:

```R
clusters <- iris %>% 
    select(-Species) %>% 
    kmeans(centers = 3)
```

We need to specify k, the number of centers in the parameters to `kmeans()`, and we choose three. We know that there are three species, so this is a natural choice. Life isn’t always that simple, but here it is the obvious choice.

The function returns an object with information about the clustering. The two most interesting pieces of information are the centers, the variable centers and the cluster assignment, the variable cluster.

```R
clusters$centers

  Sepal.Length Sepal.Width Petal.Length Petal.Width
1     5.175758    3.624242     1.472727   0.2727273
2     4.738095    2.904762     1.790476   0.3523810
3     6.314583    2.895833     4.973958   1.7031250
```

The centers are simply vectors of the same form as the input data points. They are the center of mass for each of the three clusters we have computed.

The cluster assignment is simply an integer vector with a number for each data point specifying which cluster that data point is assigned to:

```R
clusters$cluster %>% head

[1] 1 2 2 2 1 1
```

There are 50 data points for each species so if the clustering perfectly matched the species we should see 50 points for each cluster as well. The clustering is not perfect, but we can try plotting the data and see how well the clustering matches the species class.

```R
iris %>%
cbind(Cluster = clusters$cluster) %>%
ggplot() +
geom_bar(aes(x = Species, fill = as.factor(Cluster)),
position = "dodge") + scale_fill_discrete("Cluster")
```


### Hierarchical Clustering
Hierarchical clustering is a technique you can use when you have a distance matrix of your data. Here the idea is that you build up a tree structure of nested clusters by iteratively merging clusters. You start with putting each data point in their own singleton clusters. Then iteratively you find two clusters that are close together and merge them into a new cluster.


First lets define some methods for calculation distance between points, and what method we want to use for clustering:
```R
dist_methods <- c("euclidean", 
                  "maximum", 
                  "manhattan", 
                  "canberra", 
                  "binary", 
                  "minkowski")

clustering_methods <- c("ward.D", 
                       "ward.D2", 
                       "single", 
                       "complete", 
                       "average", 
                       "mcquitty", 
                       "median", 
                       "centroid")
```

In this example, we will use `euclidean` dist_method, and a `complete` clustering method.

We start by calculating the distance:
```R
dm <- 1 #euclidean
cm <- 4 #complete

iris_dist <- iris %>% 
    select(-Species) %>% 
    scale %>% dist(method = dist_methods[dm])
```

Next, we create the clusters with the `hclust` function specifying the method we selected above.
```R
clustering <- hclust(iris_dist, method = clustering_methods[cm])
```

Lets plot the result of the operation.
```R
plot(clustering, labels=iris$Species)
```

![](https://i.imgur.com/i61UqQd.png)


Lets create 3 clusters and see what data is put into what clusters
```R
clusters <- clustering %>% cutree(k = 3) 
rect.hclust(clustering, border = "red", k = 3)
```

![](https://i.imgur.com/OXZ3ZIe.png)


We can also look at the clustered data against actual data in a table format.
```R
iris2 = iris %>% 
  mutate(cluster = clusters)

table(iris2$Species, iris2$cluster)

#              1  2  3
#  setosa     49  1  0
#  versicolor  0 21 29
#  virginica   0  2 48
```


## References
Mailund, Thomas. Beginning Data Science in R. Apress, 2017.
