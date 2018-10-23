# INS300-1 LAB4

## Agenda
* [Data Already in R](#data-already-in-r)
* [Quickly Reviewing Data](#quickly-reviewing-data)
* [Reading Data](#reading-data)
* [Manipulating Data with dplyr](#manipulating-data-with-dplyr)
* [Tidying Data with tidyr](#manipulating-data-with-dplyr)
* [Exercises](#exercises)

## Data Already in R

Distributed together with R is the package dataset. You can load the package into R using the library() function and get a list of the datasets in it, together with a short description of each, like this:

```R
library(datasets)
library(help = "datasets")
```

To load an actual dataset into R’s memory, use the data() function. The datasets are all relatively small, so they are ideal for quickly testing the code you are working with.

```R
data(cars)
head(cars)
```

Furthermore, another important package used by the book is the `mlbench` package (Machine Learning Benchmark Problems). The packages are convenient for for giving examples, and if you are developing new functionality for R they are suitable for testing.

```R
install.packages("mlbench")
library(mlbench)
library(help = "mlbench")
```

## Quickly Reviewing Data

**HEAD** Shows the first *n* lines of data
```R
cars %>% head(3)
```

**TAIL** Shows the last *n* lines of data
```R
cars %>% tail(3)
```

**SUMMARY** is a generic function used to produce result summaries of the results of various model fitting functions. The function invokes particular methods which depend on the class of the first argument.


```R
data(iris)
iris %>% summary
```

## Reading Data
There are several packages for reading data in different file formats, from `Excel` to `JSON` to `XML` and so on. If you have data in a particular format, try to Google for how to read it into R. If it is a standard data format, the chances are that there is a package that can help you. 

There are various functions to read data and the most common ones are `read.csv()` and `read.table()`. These functions operate in the same way, but `read.table()` assumes the that the file is space separated, while `read.csv()` assumes that the file is comma separated.

The most common options for these functions are:

```R
header           # Boolean. Use first line of file as header
col.names        # Vector of strings. Optipn to give the dataset a header
```


*Reading data:*
```R
#Local file
dummy <- read.csv("~/path/dummy.csv", header = T)

#Online file
dummy <- read.csv(
    "http://example.com/dummy.csv", 
    header = F, 
    col.names = c("Fruit", "Amount"))
```

### Exercise


url: http://tinyurl.com/kw4xtts
Store the data in a variable and display the 10 first and last lines of the csv file. 
**note:** The csv does not include a header
<details>
<summary>Click for solution</summary>
<p>

```R
csv <- read.csv("http://tinyurl.com/kw4xtts", header = F)
head(csv, 10)
tail(csv, 10)
```
</p>
</details>


### Formatting Data
One common way to format data is using the `ifelse()`.
```R
overSpeedLimit <- . %>% { ifelse(. > 20, "over", "under") }
overSpeedLimit(cars$speed) %>% table
```

**Expected output:**
```
 over under 
    7    43
```

behind the scenes we have created a vector of all records contaning one of two values `over` or `under`. This result is then piped into the table function whish summarizes all the equal values and displays them in a table form.

To see the result with out the table we run:

```R
overSpeedLimit <- . %>% { ifelse(. > 20, "over", "under") }
overSpeedLimit(cars$speed)
```

**Expected output:**
```
 [1] "under" "under" "under" "under" "under" "under" "under" "under"
 [9] "under" "under" "under" "under" "under" "under" "under" "under"
[17] "under" "under" "under" "under" "under" "under" "under" "under"
[25] "under" "under" "under" "under" "under" "under" "under" "under"
[33] "under" "under" "under" "under" "under" "under" "under" "under"
[41] "under" "under" "under" "over"  "over"  "over"  "over"  "over" 
[49] "over"  "over"
```


## Manipulating Data with dplyr
<!-- 
select()
starts_with()
ends_with()
contains()
matches()
-->

Data frames are ideal for representing data where each row is an observation of different parameters you want to fit in models. Nearly all packages that implement statistical models or machine learning algorithms in R work on data frames. But to actually manipulate a data frame, you often have to write a lot of code to filter data, rearrange data, and summarize it in various ways. **_The `dplyr` package provides a number of convenient functions that let you modify data frames in various ways and string them together in pipes using the %>% operator_**


The dplyr package has several representations. The representation that is used in the book is the tbl_df representation. It's used because the author prefer the output when printing such tables. Feel free to use other representations if you like.


### select()

You can use it to pick out a single column:
```R
iris %>% tbl_df %>% select(Petal.Width) %>% head(3)
```

Or pick several columns:
```R
iris %>% tbl_df %>%
    select(Sepal.Width, Petal.Length) %>% head(3)
```

You can even give it ranges of columns
```R
iris %>% tbl_df %>%
    select(Sepal.Length:Petal.Length) %>% head(3)
```

There are different ways to pick columns based on the column names:
```R
iris %>% tbl_df %>%
    select(starts_with("Petal")) %>% head(3)
 
iris %>% tbl_df %>%
    select(ends_with("Width")) %>% head(3)
 
iris %>% tbl_df %>%
    select(contains("etal")) %>% head(3)
    
iris %>% tbl_df %>%
    select(matches(".t.")) %>% head(3)
```

The `matches` function searches for a **_regular expression_**, and in this example it will select any name that contains a t except if it is the first or last letter.

You can also use select() to remove columns. The previous examples select the columns you want to include, but if you use - before the selection criteria, you will exclude, instead of include, the columns you specify

```R
iris %>% tbl_df %>%
    select(-starts_with("Petal")) %>% head(3)
```

### mutate()
The mutate() function lets you add a column to your data frame by specifying an expression for how to compute it:
```R
iris %>% tbl_df %>%
    mutate(Petal.Width.plus.Length = Petal.Width + Petal.Length) %>%
    select(Species, Petal.Width.plus.Length) %>%
    head(3)
```

You can add more columns than one by specifying them in the mutate() function:
```R
iris %>% tbl_df %>%
    mutate(Petal.Width.plus.Length = Petal.Width + Petal.Length,
    Sepal.Width.plus.Length = Sepal.Width + Sepal.Length) %>%
    select(Petal.Width.plus.Length, Sepal.Width.plus.Length) %>%
    head(3)
```

### transmute()
The transmute() function works just like the mutate() function, except it is combined with a select() so the result is a data frame that only contains the new columns you make. 

_mutate() adds new variables and preserves existing; transmute() drops existing variables._

```R
iris %>% tbl_df %>%
    transmute(Petal.Width.plus.Length = Petal.Width + Petal.Length) %>%
    head(3)
```

### arrange()
The arrange() function just reorders the data frame by sorting columns according to what you specify:

```R
iris %>% tbl_df %>%
    arrange(Sepal.Length) %>%
    head(3)
```
By default, it orders numerical values in increasing order, but you can ask for decreasing order using the
desc() function:

```R
iris %>% tbl_df %>%
    arrange(desc(Sepal.Length)) %>%
    head(3)
```

### filter()
The filter() function lets you pick out rows based on logical expressions. You give the function a predicate,
specifying what a row should satisfy to be included

```R
iris %>% tbl_df %>%
    filter(Sepal.Length > 5) %>%
    head(3)
```

You can get as inventive as you want here with the logical expressions:

```R
iris %>% tbl_df %>%
    filter(Sepal.Length > 5 & Species == "virginica") %>%
    select(Species, Sepal.Length) %>%
    head(3)
```

### group_by()
The group_by() function tells dplyr that you want to work on data separated into different subsets. By itself, it isn’t that useful. It just tells dplyr that, in future computations, it should consider different subsets of the data as separate datasets.

```R
iris %>% tbl_df %>% group_by(Species) %>% head(3)
```

### summarise/summarize()
The summarise() function is used to compute summary statistics from your data frame. It lets you compute different statistics by expressing what you want to summarize. For example, you can ask for the mean of values:

```R
iris %>%
    summarise(Mean.Petal.Length = mean(Petal.Length),
    Mean.Sepal.Length = mean(Sepal.Length))
```

Where it is really powerful is in the combination with group_by(). There you can split the data into different groups and compute the summaries for each group:

```R
iris %>%
    group_by(Species) %>%
    summarise(Mean.Petal.Length = mean(Petal.Length))
```

A summary function worth mentioning here is `n()`, which just counts how many observations you have in a subset of your data:

```R
iris %>%
    summarise(Observations = n())
```

Again, this is more interesting when combined with group_by():

```R
iris %>%
    group_by(Species) %>%
    summarise(Number.Of.Species = n())
```

You can combine summary statistics simply by specifying more than one in the summary() function:

```R
iris %>%
    group_by(Species) %>%
    summarise(Number.Of.Samples = n(),
    Mean.Petal.Length = mean(Petal.Length))
```

## Tidying Data with tidyr
Tidy data means that we can plot or summarize the data efficiently. It mostly comes down to which data is represented as columns in a data frame and which is not. In practice, this means that I have columns in my data frame that I can work with for the analysis I want
to do

installing and using `tidyr`:

```R
install.packages("tidyr")
library(tidyr)
```

The paackage comes with a function, gather(), that modifies a data frame so columns become names in a factor and other columns become values.

What it does is essentially transform the data frame so that you get one column containing the name of your original columns and another column containing the values in those columns.

When gathering variables, we need to provide the name of the new key-value columns to create. The first argument, is the name of the key column, which is the name of the variable defined by the values of the column headings. The second argument is the name of the value column. The third argument defines the columns to gather.

```R
iris %>% 
    gather(key = Attribute, value = Measurement, Sepal.Length, Sepal.Width) %>%
    select(Species, Attribute, Measurement) %>% 
    head(3)
```

**Expected output:**
```
  Species    Attribute Measurement
1  setosa Sepal.Length         5.1
2  setosa Sepal.Length         4.9
3  setosa Sepal.Length         4.7
```

## Exercises

### Importing Data
To get a feeling of the steps in importing and transforming data, you need to try it yourself. So try finding a dataset you want to import. You can do that from one of the repositories I listed in the Introduction:
* RDataMining.com (http://www.rdatamining.com/resources/data)
* UCI machine learning repository (http://archive.ics.uci.edu/ml/)
* KDNuggets (http://www.kdnuggets.com/datasets/index.html)
* Reddit r/datasets (https://www.reddit.com/r/datasets)
* GitHub awesome public datasets (https://github.com/caesar0301/awesomepublic-datasets)

Or maybe you already have a dataset you would like to analyze.
Have a look at your dataset and figure out which import function you need. You might have to set a few parameters in the function to get the data loaded correctly, but with a bit of effort, you should be able to. For column names, you should either choose some appropriate ones from reading the data description or if you are loading something in that is already in mlbench, you can cheat as I did in the examples.

### Using dplyr
Now take the data you just imported and examine various summaries. It is not so important what you look at in the data, as it is that you try to summarize different aspects of it. We will look at proper analyses later. For now, just use dplyr to explore your data.

### Using tidyr
Look at the dplyr example in the book. Find the example of plotted `Sepal.Length` and `Sepal.Width` for each species. Do the same thing for `Petal.Length` and `Petal.Width`. If there is something similar you can do with the dataset you imported in the first exercise, try doing it with that dataset as well.
## References
Mailund, Thomas. Beginning Data Science in R. Apress, 2017.
