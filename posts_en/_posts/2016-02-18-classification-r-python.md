---
layout: post
title: Classification in R and Python
---

Data set *iris* which contains 150 samples with 4 morphological variables in three species of iris is wildly used by data scientists to demonstrate certain machine learning algorithm. I use k-nearest neighbors method<sup>\[[1](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm)\]</sup> to classify the species based on their sepal length and width and petal length and width.

<!-- more -->

## R
{% highlight r linenos %}
# Load data
data(iris)

# Relationships between the variables (Fig. 1)
pairs(iris[1:4],
    col=c("red","green","blue")[iris$Species],
    pch=16,
    main="Iris by Species",
    upper.panel=NULL)

legend(0.75,0.85,
    legend=levels(iris$Species),
    pch=16,
    col=c("red","green","blue"),
    cex=0.8,
    title=expression(bold("Legend")),
    y.intersp=0.6)

# Split the data into training (70%) and test (30%) data set
set.seed(1234)
size <- nrow(iris)
trainIndex <- sample(1:size, 0.7*size)
trainD <- iris[trainIndex,]
testD <- iris[-trainIndex,]

# k-Nearest Neighbor Classification, build model 
# on training data and predict on the test data
library(class)
testPred <- knn(train=trainD[,c(1:4)],
            test=testD[,c(1:4)],
            cl=trainD[,5],          # true classifications of the training data
            k=3)                    # I use k=3 here

# Performance
table(testD[,5],testPred)
##             testPred
##              setosa versicolor virginica
##   setosa         11          0         0
##   versicolor      0         20         1
##   virginica       0          1        12

# Accuracy
sum(diag(table(testD[,5],testPred)))/nrow(testD)
## [1] 0.9555556

{% endhighlight %}

<img src="/images/iris_pairs.png" alt="fig.1">
**Figure 1** Pairwise relationships between variables.
<br>
<br>

## Python 2.7
{% highlight python linenos %}
# Load data
from sklearn import datasets
iris = datasets.load_iris()
label = iris.target
data = iris.data

# Create training and test data set
import random
random.seed(1234)
size = data.shape[0]
trainIndex = random.sample(range(0,size),int(0.7*size))
testIndex = list(set(range(0,size))-set(trainIndex))

trainD = data[trainIndex]
trainL = label[trainIndex]

testD = data[testIndex]
testL = label[testIndex]

# k-Nearest Neighbor Classification
from sklearn.neighbors import KNeighborsClassifier
knn = KNeighborsClassifier(n_neighbors=3)
knn.fit(trainD,trainL)

testPred = knn.predict(testD)

# Performance
sum(testL==testPred)/float(len(testL)) # or knn.score(testD,testL)
## 0.9555555555555556

{% endhighlight %}

<br>
If you have a variable whose values are quite larger comparing with other variables, you should normalize the variables first. 

And try many times to choose the best value for k.