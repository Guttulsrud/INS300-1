# INS300-1 LAB 7

## Agenda
- [Machine Learning](#machine_learning)
- [Supervised vs Unsupervised learning](#Supervised-vs-Unsupervised-learning)
- [Today: Supervised learning](#today:-supervised-learning)
    - [Regression vs Classification](#regression-vs-classification)
- [Specifying Models](#specifying-models)
    - [Linear regresson](#linear-regresson)
        - Creating a linear regression model using the cars dataset
        - Creating a polynomial model using the cars dataset
- [Evaluating Regression Models](#Evaluating-Regression-Models)

## Libraries used in this workshop
```R
library(magrittr)
library(ggplot2)
library(dplyr)
library(tidyr)
```


## Machine Learning
Machine learning is the discipline of developing and applying models and algorithms for learning from data. 

![](https://i.imgur.com/ESV5ZCO.png)



## Supervised vs Unsupervised learning

![](https://i.imgur.com/3WEtWq2.png)

## Today: Supervised learning


### Regression vs Classification
There are two types of supervised learning: regression and classification. Regression is used when the output variable we try to target is a number. Classification is used when we try to target categorical variables.

![](https://i.imgur.com/nCLVYEq.png)


## Specifying Models
The general pattern for specifying models in R is using what is called “formulas”. The simplest form is `y ~ x`, which we should interpret as saying `y = f (x)`. 

### Linear regresson
For an example, we can use the built-in dataset cars, which just contains two variables, speed and breaking distance, where we can consider speed the x value and breaking distance the y value.

```R
cars %>% head
##   speed dist
## 1     4    2
## 2     4   10
## 3     7    4
## 4     7   22
## 5     8   16
## 6     9   10
```

#### Creating a linear regression model using the cars dataset
If we plot the dataset, we see that there is a very clear linear relationship between speed and distance. The result can be seen in the picture below.

```R
cars %>% 
    ggplot(aes(x = speed, y = dist)) + 
    geom_point() +
    geom_smooth(method = lm) #Note Method is set to lm.
```

![](https://i.imgur.com/CT9dPAh.png)


In the code below, we create a linear regression model using the `lm()` function in R. Furthermore, we use the `predict()` function to predict output values based on our model with a given `data.frame()` of parameters.

```R
fit <- cars %>% lm(dist ~ speed, data=.)
df <- data.frame(speed=c(10,20,30,40,50, 60,70))
predict(fit, df)

## Expected output:
       1         2         3         4         5         6         7 
21.74499  61.06908 100.39317 139.71726 179.04134 218.36543 257.68952 
```

To actually fit the data and get information about the fit, we use the `lm()` function with the model specification, dist ~ speed, and we can use the `summary()` function to see information about the fit:

```R
cars %>% lm(dist ~ speed, data = .) %>% summary

##Expected output

## Call:
## lm(formula = dist ~ speed, data = .)
##
## Residuals:
##     Min      1Q  Median      3Q     Max
## -29.069  -9.525  -2.272   9.215  43.201
##
## Coefficients:
## Estimate Std. Error t value Pr(>|t|)
## (Intercept) -17.5791
## speed         3.9324
##
## (Intercept) *
## 6.7584  -2.601   0.0123
## 0.4155   9.464 1.49e-12
## speed       ***
## ---
## Signif. codes:
## 0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
##
## Residual standard error: 15.38 on 48 degrees of freedom
## Multiple R-squared:  0.6511, Adjusted R-squared:  0.6438
## F-statistic: 89.57 on 1 and 48 DF,  p-value: 1.49e-12
```

Or we can use the `coefficients()` function to get the point estimates and the `confint()` function to confidence intervals for the parameters:

```R
coef <- cars %>% lm(dist ~ speed, data = .) %>% coefficients 
coef

##Expected output:

## (Intercept) speed
## -17.579095 3.932409
```

```R
conf <- cars %>% lm(dist ~ speed, data = .) %>% confint
conf

##Expected output:

##                  2.5 %    97.5 %
## (Intercept) -31.167850 -3.990340
## speed         3.096964  4.767853
```

#### Creating a polynomial model using the cars dataset
We can create a polynomial model to create a more dynamic model based on our input data. We do this by adding speed squared in order to get a  second-degree polynomial. Furthermore we can do qubed speed to create a third-degree polynomial.

For now though, we will start by creating a second-degree polynomial:

```R
cars %>%
    model.matrix(dist ~ speed + I(speed^2), data = .) %>% 
    head
    
  (Intercept) speed I(speed^2)
1           1     4         16
2           1     4         16
3           1     7         49
4           1     7         49
5           1     8         64
6           1     9         81
```

Now our model matrix has three columns, which is precisely what we want. We can fit the polynomial using the linear model function like this:

```R
cars %>% lm(dist ~ speed + I(speed^2), data = .) %>% summary
```

Or we can plot it like this:

```R
cars %>% ggplot(aes(x = speed, y = dist)) + 
    geom_point() +
    geom_smooth(method = "lm", formula = y ~ x + I(x^2))
```

To add more complex polynomial models, we can add degrees of compexity:

```cars %>% ggplot(aes(x = speed, y = dist)) + 
  geom_point() +
  geom_smooth(method = "lm", formula = y ~ x + I(x^2), se=FALSE) +
  geom_smooth(method = "lm", formula = y ~ x + I(x^2) + I(x^3), aes(color = "red"), se=FALSE)
```

### Evaluating Regression Models
```R
line <- cars %>% lm(dist ~ speed, data = .)
squared_poly <- cars %>% lm(dist ~ speed + I(speed^2), data = .)
qubed_poly <- cars %>% lm(dist ~ speed + 
    I(speed^2) +
    I(speed^3), data = .)
```

```R
predict(line, cars) %>% head
## 1 2 3 4 5 ## -1.849460 -1.849460 9.947766 9.947766 13.880175 ## 6
## 17.812584

predict(squared_poly, cars) %>% head
## 1 2 3 4 5 ## 7.722637 7.722637 13.761157 13.761157 16.173834 ## 6
## 18.786430

predict(qubed_poly, cars) %>% head
## 1 2 3 4 5 ## 7.722637 7.722637 13.761157 13.761157 16.173834 ## 6
## 18.786430
```


But how de we know which model is the best predictor?
Well, one way to evaluate the accuracy of the models is using RMSE(Root Mean Squared Error):
![](https://i.imgur.com/YJQe3Fu.png)

Creating a RMSE function in R is as easy a
```R
rmse <- function(x,t) sqrt(mean(sum((t - x)^2)))
```
Cheking the RMSE of of our predicted data:
```R
rmse(predict(line, cars), cars$dist) ## [1] 106.5529
rmse(predict(squared_poly, cars), cars$dist) ## [1] 104.0419
rmse(predict(qubed_poly, cars), cars$dist) ## [1] 
```

# Now, wouldn't this mean that more complex models is better? Well.. not necessarily.
By adding different levels of complexity to our regression, we are essentially fitting the model to our dataset. The problem this creates is called `overfitting`, which means that our model is extremly accurate towards the training data, but will not predict well when adding new data. 

So, how do we solve this problem? It's considered good practice when handling machine learning algorithms to split the dataset into two parts. One which contains traning data, and one which testing data.

```R
training_data <- cars[1:25,]
test_data <- cars[26:50,]
```

The reason for doing this is so that we can fit our model towards the a sample set of our data, and test the model using another sample set. This is usually called training data and test data.

Now if we evauluate the new models using data from the test-data in the RMSE function. We get the following output:

```R
rmse(predict(line, test_data), test_data$dist) 
## [1] 83.43421

rmse(predict(squared_poly, test_data), test_data$dist) 
## [1] 80.64634

rmse(predict(qubed_poly, test_data), test_data$dist) 
## [1] 79.20945
```

We are getting there, but there is still something wrong. There is more structure in my dataset than just the speed and distances. The data frame is sorted according to the distance so the training set has all the short distances and the test data all the long distances.

In general, you cannot know if there is such structure in your data. In this particular case, it is easy to see because the structure is that obvious, but sometimes it is subtler. So when you split your data into training and test data, you will want to sample data points randomly

We can use the `sample()` function to sample randomly zeros and ones:

```R
sampled_cars <- cars %>%
mutate(training = sample(0:1, nrow(cars), replace = TRUE))
sampled_cars %>% head

##   speed dist training
## 1     4    2        1
## 2     4   10        0
## 3     7    4        0
## 4     7   22        0
## 5     8   16        1
## 6     9   10        1
```

Now we need to filter the sampled data into two new `data.frame()` using the `filter()` function. 

```R
training_data <- sampled_cars %>% filter(training == 1) 
test_data <- sampled_cars %>% filter(training == 0)
```

This will result in the following datasets:

```R
training_data %>% head
##   speed dist training
## 1     4    2        1
## 2     8   16        1
## 3     9   10        1
## 4    11   28        1
## 5    12   20        1
## 6    13   26        1

test_data %>% head
##   speed dist training
## 1     4   10        0
## 2     7    4        0
## 3     7   22        0
## 4    10   18        0
## 5    10   26        0
## 6    10   34        0
```

Now we want to create new regression models based upon the new training dataset:

```R
line <- training_data %>% lm(dist ~ speed, data = .)
poly <- training_data %>% lm(dist ~ speed + I(speed^2), data = .)
```

Finally, we can test the models using RMSE with the test data:

```R
rmse(predict(line, test_data), test_data$dist) 
## [1] 80.83094

rmse(predict(squared_poly, test_data), test_data$dist) 
## [1] 73.93566

rmse(predict(qubed_poly, test_data), test_data$dist) 
## [1] 81.2045
```

