# INS300-1 LAB2

## Agenda
- [Repetition](#repetition)
- [Control Structures](#control-structures)
- [Functions](#functions)
- [Data frames](#data-frames)
- [Packages](#packages)
- [Pipelines](#pipelines)
- [Plots](#plots)
- [Excersice: Root Mean Square Error (RMSE)](#excersice-root-mean-square-error)


## Repetition
```R
# Basic math in R
1 + 5 / 3 * 2 
# Creating variables
name <- "Jhon Doe"
age  <- 50
# Creating a Vector
a <- c(1, 2, 3, 4, 5)
b <- 1:5
c <- seq(1, 10, by = 2) 
```

For more examples see and download links to R & RStudio go to [Lab 1](https://github.com/HermanBergner/INS300-1/blob/master/Lab%201/Lab%201.md).

## Control Structures
1. if, else
2. for
3. while
4. repeat & break
### if, else
To check condition in R, we use the simple yet powerful `if` statement.
```R
if(condition){
    # do something
} else{
    # do something
}
```
### for loop
The `for` loop in R allows you to iterate over items in a simple maner.
```R
for(i in 1:10){
    print(i * 2)
}
```
### while
Another way to iterate a set amount of times is with the `while` loop.
```R 
i <- 1
while(i < 10){
    # do something
    i++
}
```
### repeat & break
the `repeat` function takes no condition and iterates until `break` is used
```R
i <- 1
repeat{
    #do something
    i++    
    if(i == 100){
        break
    }
}
```

## Functions
### Getting documentation for functions
Some functions requires a lot of parameter, and remembering them all can be difficult. In RStudio, you can get documentation for any function by using `help(<name>)` or `?<name>` with you'r requested function name as a parameter. For example, to get documentation for the `sum()` function you would type `help(sum)` or `?sum`

### Creating a function
You can easily write your own functions. You use function expressions to define a function and an assignment to give a function a name. An example of this is a basic function that returns the given input value.
```R
f <- function(x) x;
```
If you want to call a function, simply use the name and give the function its parameters in parentheses.
```R
addTwoNumbers <- function(x, y) {x + y}
addTwoNumbers(5, 10)

#Expected output
[1] 15
```
```R
x <- c(1, 3, 5)
y <- c(2, 4, 6)
summarizeTwoVectors <- function(a, b) {
    sumOfVectorA <- sum(a)
    sumOfVectorB <- sum(b)
    sumOfVectorA + sumOfVectorB
}
summarizeTwoVectors(x, y)

#Expected output
[1] 21
```




### Task 1: Create a function that substracts two numbers
<details>
<summary>Click for solution</summary>
<p>

```R
substractTwoNumbers <- function(x, y) {x - y}
substractTwoNumbers(10, 5)

#Expected output
[1] 5
```
</p>
</details>

### Task 2: Create a function that returns the mean of a given Vector
You are given the Vector `[1, 2, 3, 4, 5]`.
Write a function that returns the mean/average of the vector without using the `mean()` function.
<details>
<summary>Need a tip?</summary>
<p>


```R
vector <- c(1, 2, 3, 4, 5)

# Getting the length of a Vector
length(vector)

#Expected output
[1] 5

# Getting the sum of a Vector
sum(vector)

#Expected output
[1] 15
```

</p>
</details>

<details>
<summary>Click for solution</summary>
<p>

```R
vector <- c(1, 2, 3, 4, 5)

average <- function(vector){
    sumOfVector <- sum(vector)
    lengthOfVector <- length(vector)
    sumOfVector / lengthOfVector
}

average(vector)

#Expected output
[1] 5
```
</p>
</details>

## Data Frames
### What is a data frame? (`?data.frame`)
The function `data.frame()` creates data frames, tightly coupled collections of variables which share many of the properties of matrices and of lists, used as the fundamental data structure by most of R's modeling software. 

### Creating a data frame
```R
df <- data.frame(1:5, 5:1)

#Expected output
  X1.5 X5.1
1    1    5
2    2    4
3    3    3
4    4    2
5    5    1
```

### Defining column names
```R
df <- data.frame(
    fruit = c("banana", "apple", "grape"), 
    amount = c(10, 15, 9)
    )

#Expected output
   fruit amount
1 banana     10
2  apple     15
3  grape      9
```

### Extracting data from of a data frame
Given the data frame:

| fruit   | amount |
|---------|--------|
| banana  |   10   |
| apple   |   15   |
| grape   |   9    |

```R
# df[<row>, <column>]

df or df[,]     # All rows and columns
df[1,]          # First row and all columns
df[1:2,]        # First two rows and all columns
df[,1]          # First column and all rows
df[2, 1]        # Second row and first row 

# Getting columns by name

df$fruit
#Expected output
[1] banana apple  grape 

df$amount
#Expected output
[1] 10 15  9
```


### Task 1: Create a data frame
Create a data frame that consists of the following data.

| Name   | Age | Height |
|--------|-----|--------|
| Joe    | 26  | 172    |
| Peter  | 32  | 185    |
| Hannah | 19  | 164    |

<details>
<summary>Click for solution</summary>
<p>

```R
name   = c("Joe", "Peter", "Hannah")
age    = c(26, 32, 19)
height = c(172, 185, 164)

df = data.frame(name, age, height)

#Expected values
    name age height
1    Joe  26    172
2  Peter  32    185
3 Hannah  19    164
```
</p>
</details>

## Packages
### What is a package in R?

### How to download and install new packages in R
Use the following command in the _RStudio Console_ to download and install a new package:
    `install.packages(<package-name>)`
Furthermore, use the following command to add the package to your current library
`library(<package-name>)`

### Task: Download and install the _magrittr_ package to your library
<details>
<summary>Click for solution</summary>
<p>

```R
install.packages("magrittr")
library(magrittr)
```
</p>
</details>

## Pipelines
Most data analysis consists of reading in some data, performing some operations on that data and, in the process, transforming it from its raw form into something we can start to make meaning out of. Then you do some summarizing or visualization toward the end.

These steps in an analysis are typically expressed as a sequence of function calls that each change the data from one form to another. See example below.

```R
data <- readDataFromDisk("~/data.csv")
sanitized <- removeBadStuff(data)
summarized <- summarizeAllTheValues(sanitized)
plotOnlyTheCoolData(summarized)
```
The code above will usually be written like this:
```R
plotOnlyTheCoolData(
    summarizeAllTheValues(
        removeBadStuff(
            readDataFromDisk("~/data.csv")
        )
    )
)
```
There isn’t really anything wrong with writing a data analysis in this way. But there are typically many more steps involved than just these. The `magrittr` package implements a trick to alleviate this problem. It does this by introducing a “pipe operator,” `%>%`, that lets you write the functions you want to combine from left to right
```R
readDataFromDisk("~/data.csv") 
%>% removeBadStuff()
%>% summarizeAllTheValues()
%>% plotOnlyTheCoolData()
```

### How to use pipelines
```R
1:10 %>% sum()

#Expected output
[1] 55


1:5 %>% sum() %>% cos() %>% abs() %>% sqrt()

#Expected output
[1] 0.8716008
```

### The magical `.`
Now, you cannot always be so lucky that all the functions you want to call in a pipeline take the left side of the %>% as its first parameter. If this is the case, you can still use the function, though, because magrittr interprets . in a special way. If you use . in a function call in a pipeline, then that is where the left side of the %>% operation goes instead of as default first parameter of the right side. So if you need the data to go as the second parameter, you put a `.` there, since `x %>% f(y, .)` is equivalent to `f(y, x)`.

```R
data.frame(x = 1:5, y = 5:1) %>% plot(y ~ x, data = .)
```

### Task: Pipelines
Convert the following to a pipeline.
```R
sqrt(cos(sin(sum(abs(-10:5)))))

#Expected output
[1] 0.8456931
```

<details>
<summary>Click for solution</summary>
<p>

```R
-10:5 %>% abs() %>% sum() %>% sin() %>% cos() %>% sqrt()
```
</p>
</details>



## Plots
### What is a plot?
Plots is used to visualize our data into informative graphs. To do this, we use the plot function, a generic function for plotting R objects. 

### How to use plots
In the following example we want to visualize the cost of fruit from our fruit dataset in a barchart.

```R
#Declare Variables
fruit <- c("banana", "apple", "grape")
amount <- c(10, 15, 9)

#Create data frame based upon variables
df <- data.frame(fruit, amount)

#Visualize the cost of the fruit using the barplot
barplot(df$amount, names.arg = df$fruit, col=c("yellow", "red", "purple"))
```
Expected result:
![](https://i.imgur.com/rg15EhP.png)


### Task: Visualize a given dataset using plot

Create a dataframe based upon the following data, and visualize the height of Joe, Peter and Hannah in a barchart.

| Name   | Age | Height |
|--------|-----|--------|
| Joe    | 26  | 172    |
| Peter  | 32  | 185    |
| Hannah | 19  | 164    |

<details>
<summary>Click for solution</summary>
<p>

```R
name <- c("Joe", "Peter", "Hannah")
age <- c(26, 32, 19)
height <- c(172, 185, 164)

df <- data.frame(name, age, height)

barplot(df$height, names.arg = df$name, ylab = "Height in CM")

```
Expected result:
![](https://i.imgur.com/jkVpnCq.png)
</p>
</details>

## Excersice Root Mean Square Error
If you have “true” values, t = (t1, ..., tn) and “predicted” values y = (y1, ..., yn), then the root mean square error is defined as: 

![](https://i.imgur.com/ULnyKsM.png)

You are given the following dataset:

```R
x = 1, 2, 2, 3
t = 1, 2, 3, 6

# Formula for linear regression for given dataset
y = 2.5 * x - 2
```
Write a pipeline that computes `RMSE` from a data frame containing the `t` and `y` _values_. Remember that you can do this by first computing the square difference in one expression, then computing the `mean()` of that in the next step, and finally computing the square root of this. The R function for computing the square root is `sqrt()`.

<details>
<summary>Tips</summary>
<p>

```R
x <- c(1, 2, 2, 3)
t <- c(1, 2, 3, 6)

model <- . %>% { 2.5 * . - 2 }

df <- data.frame(y = model(x), t)
```

```R
# Getting the average/ mean
mean()

# Getting the square root
sqrt()
```

</p>
</details>

## References
Mailund, Thomas. Beginning Data Science in R. Apress, 2017.
