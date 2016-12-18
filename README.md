## 1.Introduction
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. The goal of this project is to predict the manner in which they did the exercise. This is the "classe" variable in the training set.

## 2.Data Preparation
Read the data
```r
> trainUrl <-"https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
> testUrl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
> training.raw <- read.csv(url(trainUrl), na.strings=c("", "NA", "#DIV/0!"))
> testing.raw <- read.csv(url(testUrl), na.strings=c("", "NA", "#DIV/0!"))
```
Dimension of the data
```r
> dim(training.raw)
[1] 19622   160
> dim(testing.raw)
[1]  20 160
```
Data cleaning: remove data with large portion of missing values.
```r
> removeIndx<-which(colSums(is.na(training.raw))>(ncol(training.raw)*0.7))
> train.clean <- training.raw[,-c(removeIndx)]
> test.clean <- testing.raw[,-c(removeIndx)] 
> myData <- train.clean[,8:60]
> testing <- test.clean[,8:60]
> dim(myData)
[1] 19622    53
```
Data partition: split the cleaned raw training data set into two parts: 70% for training and 30% for validation.
```r
> set.seed(12345)
> inTrain <- createDataPartition(myData$classe, p = 0.7)[[1]]
> training <- myData[ inTrain,]
> valid <- myData[-inTrain,]
> dim(training)
[1] 13737    53
> dim(valid)
[1] 5885   53 
```
## 3.Modeling
### 3.1 Random Forest
### 3.2 Decision Tree
### 3.3 Ensemble
## 4. Prediction on Test Data Set
