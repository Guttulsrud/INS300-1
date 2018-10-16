# INS300-1 LAB5

## Agenda
* [Basic Graphics](#basic-graphics)
* [ggplot2](#ggplot2)
    * [Using-qplot()](#using-qplot())
* [Using Geometries](#using-geometries)
    * [Cars Dataset Examples](#cars-dataset-examples)
    * [Iris Dataset Examples](#iris-dataset-examples)
    * [Advanced Graphs](#advanced-graphs)
    * [ggplot and tidyr](#ggplot-and-tidyr)
* [Facets](#facets)
* [Scaling](#scaling)
* [Themes and Other Graphics Transformations](#themes-and-other-graphics-transformations)
* [Figures with Multiple Plots](#figures-with-multiple-plots)
* [References](#references)

## Basic Graphics
Creating a plot from x, y values is fairly straight forward.
```R
x <- rnorm(50) 
y <- rnorm(50) 
plot(x, y)
```

or using a dataset such as the `cars` dataset
```R
cars %>% plot(dist ~ speed, data = .)
```

Adding a linear model to the cars dataset
```R 
cars %>% 
plot(dist ~ speed, data = .)

cars %>%
lm(dist ~ speed, data = .) %>%
abline(col = "red")
```

## ggplot2

### Using qplot()
The `qplot()` function can be used to plot simple scatterplots the same way as the `plot()` function. To plot the cars data with `qplot()` you can do the following:

```R
cars %>% qplot(speed, dist, data = .)
```
when you use the qplot() function, it returns a ggplot object that you can use for later.
```R
plotObject <- cars %>% qplot(speed, dist, data = .)
```

When you’re using qplot(), some transformations of the plotting object are done before qplot() returns the object. qplot() is guessing at what kind of plot you are likely to want and then doing transformations on a plot to get there. To get the full control of the final plot, we skip qplot() and do all the transformations explicitly.
```R
iris %>% qplot(Petal.Width, Petal.Length , color = Species, data = .)
```

### Using Geometries
By stringing together several geometry commands, you can display the same data in different ways—e.g., scatterplots combined with smoothed lines—or put several data sources on the same plot.


#### Cars Dataset Examples
You start with the scatterplot for cars where we used the following:

```R
cars %>% qplot(speed, dist, data = .)
```

To create this plot using explicit geometries we want a `ggplot` object, we need to map the `speed` parameter from the data frame to the x-axis and the `dist` parameter to the y-axis, and we need to plot the data as points:

```R
ggplot(cars) + geom_point(aes(x = speed, y = dist))
```

Similar to the function above, we can do move the aesthetics to the first function. The `aes()` function defines the mapping from data to graphics just for the `geom_point()` function. 

Sometimes we want to have different mappings for different geometries, and sometimes we do not. If we want to share aesthetics between functions, we can set it in the `ggplot()` function call instead. 

```R
ggplot(cars, aes(x = speed, y = dist)) + geom_point()
```

Since ggplot() takes a data frame as its first argument, it is a typical pattern to first modify data in a string of %>% operations and then give it to ggplot() and follow that with a series of + operations.

```R
cars %>% ggplot(aes(x = speed, y = dist)) + geom_point()
```

#### Iris Dataset Examples
For the iris data, we used the following `qplot()` call to create a scatterplot with colors determined by the Species variable.
```R
iris %>% qplot(Petal.Width, Petal.Length,
color = Species, data = .)
```

The corresponding code using `ggplot()` and `geom_point()` looks like this:
```R
iris %>% ggplot +
geom_point(aes(x = Petal.Width, y = Petal.Length,
                   color = Species))
```
Instead of having the color depending on the data, you can hard code a color for the visualisation.
```R
iris %>% ggplot +
geom_point(aes(x = Petal.Width, y = Petal.Length),
               color = "red")
```

code for plotting a histogram and a density plot:
Constructed a histogram and density plot can be done with the geom_histogram() and geom_density(), respectively:

```R
cars %>% ggplot + geom_histogram(aes(x = speed), bins = 10) 
cars %>% ggplot + geom_density(aes(x = speed))
```

You can combine more geometries to display the data in more than one way. This isn’t always meaningful and depends on how data is summarized—combining scatterplots and histograms might not be so useful. However, you can, for example, make a plot showing the car speed both as a histogram and a density.

```R
cars %>%
ggplot(aes(x = speed, y = ..count..)) + 
geom_histogram(bins = 10) +
geom_density()
```
By setting y = `..count..`, you tell both geometries to use counts as the y-axis. To get densities instead, you can use y = `..density..`

```R
cars %>% 
ggplot(aes(x = speed, y = dist)) +
geom_point() +
geom_smooth(method = "lm")
```
#### Advanced Graphs
To create a linear model for a scatter plot we can use the `geom_smooth()` function found within ggplot2. If we dont provide the `geom_smooth()` function with any method it will by default use the method `loess`
```R
cars %>% 
ggplot(aes(x = speed, y = dist)) +
geom_point() +
geom_smooth()
## `geom_smooth()` using method = 'loess'
```

You can also use more than one geometry to plot more than one variable. For the longley data, you could use two different geom_line() to plot the Unemployed and the Armed.Forces data
```R
longley %>% ggplot(aes(x = Year)) + geom_line(aes(y = Unemployed)) +
geom_line(aes(y = Armed.Forces), color = "blue")
```

You can also combine geom_line() and geom_point() to get both lines and points for your data.
```R
longley %>% ggplot(aes(x = Year)) +
geom_point(aes(y = Unemployed)) +
geom_point(aes(y = Armed.Forces), color = "blue") +
geom_line(aes(y = Unemployed)) +
geom_line(aes(y = Armed.Forces), color = "blue")
```
#### ggplot and tidyr
Plotting two variables using different aesthetics like this is fine for most applications, but it is not always the optimal way to do it. The problem is that we are representing that the two measures, Unemployment and Armed.Forces, are two different measures we have per year and that we can plot together in the plotting code

A better way to do this is to use the `gather` function from the library `tidyr` as seen below:
```R
longley %>% 
gather(key, value, Unemployed, Armed.Forces) %>%
ggplot(aes(x = Year, y = value, color = key)) + 
geom_line()
```

Once you have transformed the data, you can change the plot with little extra code. If, for instance, you want the two values on different facets, we can simply specify this (instead of setting the colors)
```R
longley %>%
gather(key, value, Unemployed, Armed.Forces) %>% 
ggplot(aes(x = Year, y = value)) + 
geom_line() +
facet_grid(key ~ .)
```

### Facets
**Problem:**
You want to do split up your data by one or more variables and plot the subsets of data together. To solve this we use can use the `facet_grid` or `facet_wrap` functions found within ggplot. The result of faceting can be seen using the function below:

```R
iris %>% 
gather(Measurement, Value, -Species) %>% 
ggplot(aes(x = Species, y = Value)) +
geom_boxplot() +
facet_grid(Measurement ~ .)
```

We can plot the four measurements for each species in different facets, but they are on slightly different scales, so we will only get a good look at the range of values for the largest range. We can fix this by setting the y-axis free. 

```R
iris %>% 
gather(Measurement, Value, -Species) %>%
ggplot(aes(x = Species, y = Value)) +
geom_boxplot() +
facet_grid(Measurement ~ ., scale = "free_y")
```

The labels used for facets are taken from the factors in the variables used to construct the facet. This is a good default, but for print quality plots, you often want to modify the labels a little. You can do this using the labeller parameter to facet_grid()

You can give `labeller()` a named argument and specify a factor to make labels, with lookup tables that map the levels to the labels.

```R
label_map <- c(Petal.Width  = "Petal Width", 
               Petal.Length = "Petal Length",
               Sepal.Width  = "Sepal Width",
               Sepal.Length = "Sepal Length")

iris %>% 
    gather(Measurement, Value, -Species) %>% 
    ggplot(aes(x = Species, y = Value)) + 
    geom_boxplot() +
    facet_grid(Measurement ~ ., scale = "free_y",
        labeller = labeller(Measurement = label_map))
```


### Scaling
Geometries specify part of how data should be visualized and scales another. The geometries tell ggplot2 how you want your data mapped to visual components, like points or densities, and scales tell ggplot2 how dimensions should be visualized.

The simplest scales to think about are the x- and y-axes, where values are mapped to positions on the plot as you are familiar with, but scales also apply to visual properties such as colors.

```R
cars %>% 
ggplot(aes(x = speed, y = dist)) +
    geom_point() +
    geom_smooth(method = "lm") +
    scale_x_continuous("Speed") +
    scale_y_continuous("Stopping Distance")
```

In general, you can use the scale_x/y_ continuous() functions to control the axis graphics. For instance, to set the breakpoints shown, if you wanted to plot the longley data with a tick mark for every year instead of every five years, you could set the breakpoints to every year:

```R
longley %>% 
gather(key, value, Unemployed, Armed.Forces) %>% 
    ggplot(aes(x = Year, y = value)) + 
    geom_line() + scale_x_continuous(breaks = 1947:1962) +
    facet_grid(key ~ .)
```

To reverse an axis, use scale_x/y_reverse(). This is better than reversing the data mapped in the aesthetic since all the plotting code will just be updated to the reversed axis.

```R
cars %>% 
ggplot(aes(x = speed, y = dist)) + 
    geom_point() + 
    geom_smooth(method = "lm") + 
    scale_x_reverse("Speed") + 
    scale_y_continuous("Stopping Distance")
```

Neither axis has to be continuous, though. If you map a factor to x or y in the aesthetics, you get a discrete axis. Species is a factor, the x-axis will be discrete, and you can show the data as a boxplot and the individual data points using the jitter geometry.

```R
iris %>% 
ggplot(aes(x = Species, y = Petal.Length)) + 
    geom_boxplot() + 
    geom_jitter(width = 0.1, height = 0.1)
```

If you want to modify the x-axis, you need to use scale_x_ discrete() instead of scale_x_continuous().
You could, for instance, use this to modify the labels on the axis to put the species in capital letters:

```R
iris %>% 
ggplot(aes(x = Species, y = Petal.Length)) + 
    geom_boxplot() + 
    geom_jitter(width = 0.1, height = 0.1) + 
    scale_x_discrete(labels = c("setosa" = "Setosa",
                              "versicolor" = "Versicolor",
                              "virginica" = "Virginica"))
```

You can plot the iris measurements per species and give them a different color for each species. Since it is the boxes you want to color, you need to use the fill aesthetics. Otherwise, you would color the lines around the boxes.

```R
iris %>% 
gather(Measurement, Value, -Species) %>% 
ggplot(aes(x = Species, y = Value, fill = Species)) +
    geom_boxplot() +
    facet_grid(Measurement ~ ., scale = "free_y",
    labeller = labeller(Measurement = label_map))
```

There are different ways to modify color scales. There are two classes, as there are for axes—discrete and continuous. The Species variable in iris is discrete, so to modify the fill color, you need one of the functions for that. The simplest is to give a color per species explicitly. You can do that with the `scale_ fill_manual()` function.

```R
iris %>% 
gather(Measurement, Value, -Species) %>% 
ggplot(aes(x = Species, y = Value, fill = Species)) +
    geom_boxplot() +
    scale_fill_manual(values = c("red", "green", "blue")) + 
    facet_grid(Measurement ~ ., scale = "free_y",
        labeller = labeller(Measurement = label_map))
```

Explicitly setting colors is risky business, though, unless you have a good feeling for how colors
work together and which combinations can be problematic for color-blind people. It is better to use one of the “brewer” choices. These are methods for constructing good combinations of colors (see http:// colorbrewer2.org) and you can use them with the scale_fill_brewer() function.

```R
iris %>% 
gather(Measurement, Value, -Species) %>% 
ggplot(aes(x = Species, y = Value, fill = Species)) +
    geom_boxplot() +
    scale_fill_brewer(palette = "Greens") +
    facet_grid(Measurement ~ ., scale = "free_y",
        labeller = labeller(Measurement = label_map))
```

### Themes and Other Graphics Transformations

Most of using ggplot2 consist of specifying geometries and scales to control how data is mapped to visual components, but you also have control over how the final plot will look through functions that only concern the final visual result.

You can flip x and y with `coord_flip()`. This can of course also be achieved just by changing the aesthetics. You can control the placement of facet labels using the switch option to `facet_grid()`. Giving the switch parameter the value y will switch the location of that label.
```R
iris %>%
gather(Measurement, Value, -Species) %>%
ggplot(aes(x = Species, y = Value, fill = Species)) +
    geom_boxplot() +
    scale_x_discrete(labels = c("setosa" = "Setosa",
                              "versicolor" = "Versicolor",
                                "virginica" = "Virginica")) 
    + scale_fill_brewer(palette = "Greens") +
    facet_grid(Measurement ~ ., switch = "y",
    labeller = labeller(Measurement = label_map)) +
    coord_flip()
```

The labels look ugly with the color background. You can remove it by modifying the theme using theme(strip.background = element_blank())
```R
iris %>% 
gather(Measurement, Value, -Species) %>% 
ggplot(aes(x = Species, y = Value, fill = Species)) +
    geom_boxplot() +
    scale_x_discrete(labels = c("setosa" = "Setosa",
                              "versicolor" = "Versicolor",
                                "virginica" = "Virginica")) +
    scale_fill_brewer(palette = "Greens") +
    facet_grid(Measurement ~ ., switch = "y",
    labeller = labeller(Measurement = label_map)) +
    coord_flip() +
    theme(strip.background = element_blank()) + 
    theme(legend.position="top")
```

We want the labeled species to be in capital letters just like the axis labels. Well, you know how to do that using the labels parameter to a scale so that the final plotting code could look like this:

```R
label_map <- 
    c(Petal.Width = "Petal Width", 
    Petal.Length = "Petal Length", 
    Sepal.Width = "Sepal Width", 
    Sepal.Length = "Sepal Length") 
    
species_map <- c(setosa = "Setosa",
                versicolor = "Versicolor",
                virginica = "Virginica")
                
iris %>% gather(Measurement, Value, -Species) %>%
ggplot(aes(x = Species, y = Value, fill = Species)) +
    geom_boxplot() +
    scale_x_discrete(labels = species_map) +
    scale_fill_brewer(palette = "Greens", labels = species_map) +  
    facet_grid(Measurement ~ ., switch = "y",
    labeller = labeller(Measurement = label_map)) + 
    coord_flip() +
    theme(strip.background = element_blank()) + 
    theme(legend.position="top")
```



## Figures with Multiple Plots
The ggplot2 package doesn’t directly support combining multiple plots, but it can be achieved using the underlying graphics system, grid. Working with the basic grid you have many low-level tools for modifying graphics, but for just combining plots you want more high-level functions, and you can get them from the gridExtra package.
To combine plots, you first create them as you normally would. So, for example, you could make two plots of the iris data like this:


```R
petal <- iris %>% ggplot() +
    geom_point(aes(x = Petal.Width, y = Petal.Length, color = Species)) + 
    theme(legend.position="none")

sepal <- iris %>% ggplot() +
    geom_point(aes(x = Sepal.Width, y = Sepal.Length,
    color = Species)) + 
    theme(legend.position="none")
```

You then import the gridExtra package:
`library(gridExtra)`

You then use the grid.arrange() function to create a grid of plots, putting in the two plots you just created
`grid.arrange(petal, sepal, ncol = 2)`

## References
Mailund, Thomas. Beginning Data Science in R. Apress, 2017.
