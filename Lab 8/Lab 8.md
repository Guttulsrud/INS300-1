# LAB 8


# Supervised learning: Classification

![](https://i.imgur.com/HuwRXhp.png)

## Breast cancer prediction
Today we can use the breast cancer data from the mlbench library and ask if the clump thickness has an effect on the risk of a tumor being malignant. That is, we want to see if we can predict the Class variable from the Cl.thickness variable. 

```R
library(mlbench)
data("BreastCancer")
BreastCancer %>% head

## Id Cl.thickness Cell.size Cell.shape
## 1 1000025
## 2 1002945
## 3 1015425
## 4 1016277
## 5 1017023
## 6 1017122
##   Marg.adhesion Epith.c.size Bare.nuclei
## 1             1            2           1
## 2             5            7          10
## 3             1            2           2
## 4             1            3           4
## 5             3            2           1
## 6             8            7          10
##   Bl.cromatin Normal.nucleoli Mitoses     Class
## 1           3               1       1    benign
## 2           3               2       1    benign
## 3           3               1       1    benign
## 4           3               7       1    benign
## 5           3               1       1    benign
## 6           9               7       1 malignant
```

**Let's plot the data**

```R
BreastCancer %>%
ggplot(aes(x = Cl.thickness, y = Class))
```

**Modify the visualization using jitter**
```R
BreastCancer %>%
ggplot(aes(x = Cl.thickness, y = Class)) + 
geom_jitter(height = 0.05, width = 0.3, alpha=0.4)
```
**Expected result:**

![](https://i.imgur.com/qsmg4av.png)


**Creating a logistical regression model:**

We cannot directly fit the breast cancer data with logistic regression, though. There are two problems. The first is that the breast cancer dataset considers the clump thickness ordered factors, but for logistic regression we need the input variable to be numeric. 

```R
formatted_breastCancer <- BreastCancer %>%
  mutate(Cl.thickness.numeric = as.numeric(as.character(Cl.thickness)))
  
formatted_breastCancer %>% View
```

The second problem is that the `glm()` function expects the response variable to be numerical, coding the classes like 0 or 1, while the BreastCancer data encodes the classes as a factor.


```R
formatted_breastCancer <- formatted_breastCancer %>%
  mutate(IsMalignant = ifelse(Class == "benign", 0, 1))
  
formatted_breastCancer %>% View
```

We can translate the input variable to numerical values and the response variable to 0 and 1 and plot the data together with a fitted model. For the `geom_smooth()` function, we specify that the method is glm and that the family is binomial. To specify the family, we need to pass this argument on to the smoothing method, and that is done by giving the method.args parameter a list of named parameters; here we just give it `list(family = "binomial)`.


```R
formatted_breastCancer %>%
  ggplot(aes(x = Cl.thickness.numeric, y = IsMalignant)) +
  geom_jitter(height = 0.05, width = 0.3, alpha=0.4) + 
  geom_smooth(method = "glm", method.args = list(family = "binomial"))
```

![](https://i.imgur.com/yTN7kmC.png)

### Evaluating our logistic regression model

If you want to do classification rather than regression, then the root mean square error is not the function to use to evaluate your model. With classification, you want to know how many data points are classified correctly and how many are not.
As an example, we can take the breast cancer data and fit a model:

```R
fitted_model <- formatted_breastCancer %>%
  glm(IsMalignant ~ Cl.thickness.numeric, data = .)
```
To get its prediction, we can again use predict(), but we will see that for this particular model the predictions are probabilities of a tumor being malignant. By default, the model we created with glm() will be in “logit” units, but we can use the type parameter to get it in the input unit used with probabilities.

```R
predict(fitted_model, formatted_breastCancer) %>% head
        1         2         3         4         5         6 
0.4152011 0.4152011 0.1733065 0.5361484 0.2942538 0.7780430 
```

We would need to translate that into actual predictions. The natural choice here is to split the probabilities at 50%. If we are more certain that a tumor is malignant than benign, we will classify it as malignant.

```R
classify <- function(probability) ifelse(probability < 0.5, 0, 1)

classified_malignant <- classify(predict(fitted_model, formatted_breastCancer))

classified_malignant %>% head
```

**Confusion Matrix**
In any case, if we just put the classification threshold at 50/50 then we can compare the predicted classification against the actual classification using the `table()` function, as follows:

```R
table(formatted_breastCancer$IsMalignant, classified_malignant)

      0   1
  0 437  21
  1  76 165
```

This table, contrasting predictions against true classes, is known as the `confusion matrix`. The rows count how many zeros and ones we see in the `formatted_breastCancer$IsMalignant` argument and the columns how many zeros and ones we see in the `classified_malignant` argument. So the first row is where the data says the tumors are not malignant and the second row is where the data says that the tumors are malignant. The first column is where the predictions say the tumors are not malignant while the second column is where the predictions say that they are.

**_Hard to read?_**
This, of course, depends on the order of the arguments to `table()`, it doesn’t know which argument contains the data classes and which contains the model predictions. It can be a little hard to remember which dimension, rows or columns, are the predictions but you can provide a parameter, dnn (dimnames names), to make the table remember it for you.

```R
table(formatted_breastCancer$IsMalignant, 
    classified_malignant, 
    dnn=c("Data", "Predictions"))
    
# Expected Output:
    Predictions
Data   0   1
   0 437  21
   1  76 165
```

The correct predictions are on the diagonal, and the off-diagonal values are where our model predicts incorrectly.
* _The first row is where the data says that tumors are not malignant._ 
    * The first element, where the model predicts that the tumor is benign, and the data agrees, is called the **true negatives**. 
    * The element to the right of it, where the model says a tumor is malignant but the data says it is not, is called the **false positives**.
* _The second row is where the data says that tumors are malignant._ 
    * The first column is where the prediction says that it isn’t a malignant tumor, and these are called the **false negatives**. 
    * The second column is the cases where both the model and the data says that the tumor is malignant. That is the **true positives**.

The classes do not have to be zeros and ones. That was just easier in this particular model where I had to translate the classes into zeros and ones for the logistic classification anyway. But really, the classes are "benign" and "malignant", like this:

```R
classify <- function(probability) ifelse(probability < 0.5, "benign", "malignant")
classified <- classify(predict(fitted_model, formatted_breastCancer))
classified %>% head

#          1           2           3           4           5           6 
#   "benign"    "benign"    "benign" "malignant"    "benign" "malignant"
```

And displyed in a table like this:

```R
table(formatted_breastCancer$Class, classified, dnn=c("Data", "Predictions"))

#           Predictions
# Data        benign malignant
# benign       437        21
# malignant     76       165
```



## K-Nearest Neighbor (KNN)
```R
install.packages("class")
library(class)
?knn
```


```R
set.seed(1234)
randomized = sample(2, nrow(iris), replace = T, prob = c(0.8, 0.2))
iris_test = iris[randomized == 2, 1:2]
iris_train = iris[randomized == 1, 1:2]
iris_train_labels = iris[randomized == 1, "Species"]
iris_test_labels = iris[randomized == 2, "Species"]


newIris = data.frame("Sepal.Length" = 5.1,"Sepal.Width" = 4.2)


predicted <- knn(train = iris_train, test = iris_test, cl = iris_train_labels, k = 3)
table(predicted, iris_test_labels)


predicted <- knn(train = iris_train, test = iris_test, cl = iris_train_labels, k = 3)
predicted
```

## Descision tree

```R
install.packages("party")
library(party)
library(caTools)

split <- sample.split(iris$Species, SplitRatio = 0.8)
train = subset(iris, split == TRUE)
test = subset(iris, split == FALSE)
model <- ctree(Species ~ ., data = train)
table(predict(model, test), test$Species)
plot(model)
```

![](https://i.imgur.com/uy0vfuM.png)
