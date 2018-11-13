# Agenda
- **Download todays dataset:** _Black Friday Dataset from Kaggle_
-

## Libraries
```R
library(ggplot2)
library(magrittr)
library(dplyr)
library(tidyr)
library(purrr)
```

# Loading the dataset into R
Download the dataset from this link: https://www.kaggle.com/mehdidag/black-friday

```R
BlackFriday <- read.csv(file.choose())
str(BlackFriday)
BlackFriday %>% head
```

# Exploring the Dataset
## Finding the number of products bought at Black Friday
```R
BlackFriday %>%
  nrow() %>%
  paste("products bought at Black Friday")

#[1] "537577 products bought at Black Friday"
```

## Finding the number of unique buyers
```R
library(dplyr)

BlackFriday %>%
  distinct(User_ID) %>%
  nrow() %>%
  paste("buyers registered at Black Friday")

#[1] "5891 buyers registered at Black Friday"
```

## Age distribution
```R
BlackFriday %>% ggplot + geom_bar(aes(x = Age, fill = Age))
```
## Gender Distribution
```R
BlackFriday %>% ggplot + geom_bar(aes(x = Gender, fill = Gender))
```

## Gender distribution on age
```R
BlackFriday %>% ggplot +
  geom_bar(aes(x = Gender, fill = Gender)) +
  facet_wrap(. ~ Age)
```

## Price VS Age
```R
BlackFriday %>%
  group_by(User_ID, Age) %>%
  summarise(transactions = n(), total = sum(Purchase)) %>%
  ggplot(aes(x = Age, y = t, fill = Age)) +
  geom_boxplot()
```

## Price vs Gender on Age
```R
BlackFriday %>%
  group_by(User_ID, Gender, Age) %>%
  summarise(transactions = n()) %>%
  ggplot(aes(x = Gender, y = transactions, fill = Gender)) +
  geom_col() +
  facet_wrap(~Age)
```

# Supervised Learning


# Unsupervised Learning

## K-means

In k-means clustering you attempt to separate the data into k clusters, where you determine the number k. The data usually has to be in the form of numeric vectors. Strictly speaking, the method will work as long as you have a way of computing the mean of a set of data points and the distance between pairs of data points. The R function for k-means clustering, kmeans, wants numerical data.

![](https://i.imgur.com/qvKsumu.png)

The algorithm essentially works by first guessing at k “centers” of proposed clusters. Then each data point is assigned to the center it is closest to, creating a grouping of the data, and then all centers are moved to the mean position of their clusters. This is repeated until an equilibrium is reached.

![](https://i.imgur.com/LYhkvUk.gif)

This time, we will try to create clusters based upon the purchase amounts:

```R
BlackFridayPurchases <- BlackFriday %>%
  select(Purchase)

BlackFridayPurchases %>% head
```

### How many clusters should we use?
**Elbow validation method**
```R
tot_withinss <- map_dbl(1:10,  function(k){
  model <- kmeans(x = BlackFridayPurchases, centers = k)
  model$tot.withinss
})

elbow_df <- data.frame(
  k = 1:10,
  tot_withinss = tot_withinss
)

ggplot(elbow_df, aes(x = k, y = tot_withinss)) +
  geom_line() +
  scale_x_continuous(breaks = 1:10)
```

Let's use 3 clusters:
```R
model_km3 <- kmeans(BlackFridayPurchases, centers = 3)

model_km3 %>% head
```

Extracting the clusters and adding them to our BlackFriday dataset:
```R
clust_km3 <- model_km3$cluster

BlackFriday_Clust <- mutate(BlackFriday, cluster = clust_km3)

BlackFriday_Clust %>% head
```

Extracting some statistics:
```R
BlackFriday_Clust_Note <- BlackFriday_Clust %>%
  group_by(cluster) %>%
  summarise(min_purchase = min(Purchase),
            max_purchase = max(Purchase),
            avg_purchase = round(mean(Purchase),0))

BlackFriday_Clust_Note

  cluster min_purchase max_purchase avg_purchase
    <int>        <dbl>        <dbl>        <dbl>
1       1        13050        23961        17056
2       2          185         6630         4217
3       3         6631        13049         9044
```

Visualizing each cluster of purchases by cities:
```R
BlackFriday_Clust %>%
  group_by(City_Category, cluster) %>%
  summarise(n = n()) %>%
  ggplot(aes(x=City_Category, y = n)) +
  geom_col(aes(fill = as.factor(cluster))) +
  theme_linedraw() +
  theme(legend.box.background	= element_rect(colour = "black"),
        legend.background = element_rect(fill = "gainsboro"),
        panel.background = element_rect(fill = "gainsboro", colour = "white", size = 0.5, linetype = "solid"), #theme panel settings
        plot.background = element_rect(fill = "gainsboro"), #theme panel settings
        panel.grid.major = element_line(size = 0.5, linetype = 'solid', colour = "white"), #theme panel settings
        panel.grid.minor = element_line(size = 0.25, linetype = 'solid', colour = "white"), #theme panel settings
        plot.title = element_text(hjust = 0, face = 'bold',color = 'black'), #title settings
        plot.subtitle = element_text(face = "italic")) + #subtitle settings
  labs(x = 'City Category', y = 'Total Purchase (dollars)', title = "Black Friday", #name title and axis
       subtitle = "Total purchases in each cluster by city") + #name subtitle
  guides(fill=guide_legend(title = "Cluster")) + #remove color legend
  scale_y_continuous(labels = scales::comma) #prevent scientific number in x-axis
```

![](https://i.imgur.com/ssQat9C.png)
