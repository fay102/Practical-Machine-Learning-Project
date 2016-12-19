# Practical Machine Learning Project


## 1.Introduction
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. The goal of this project is to predict the manner in which they did the exercise. This is the "classe" variable in the training set.

## 2.Data Preparation

Read the data

```r
trainUrl <-"https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
testUrl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
training.raw <- read.csv(url(trainUrl), na.strings=c("", "NA", "#DIV/0!"))
testing.raw <- read.csv(url(testUrl), na.strings=c("", "NA", "#DIV/0!"))
```

Dimension of the data

```r
dim(training.raw)
## [1] 19622   160
dim(testing.raw)
## [1]  20 160
```

Data cleaning: we first remove data with large portion (more than 70%) of missing values, then pick up the predictors which are related to the response variable.

```r
removeIndx<-which(colSums(is.na(training.raw))>(ncol(training.raw)*0.7))
train.clean <- training.raw[,-c(removeIndx)]
test.clean <- testing.raw[,-c(removeIndx)] 
myData <- train.clean[,8:60]
testing <- test.clean[,8:60]
dim(myData)
## [1] 19622    53
```

Data partition: split the cleaned raw training data set into two parts: 70% for training and 30% for validation.

```r
set.seed(12345)
inTrain <- createDataPartition(myData$classe, p = 0.7)[[1]]
training <- myData[ inTrain,]
valid <- myData[-inTrain,]
dim(training)
## [1] 13737    53
dim(valid)
## [1] 5885   53 
```

## 3.Modeling
We applied Random Forest, Decision Tree and GBM to fit the training data set. For Random Forest, a 5-fold cross validation is used to determine the optimal value of mtry. For Decision Tree, we also used 5-fold cross validation to choose the best complexity parameter. In GBM, tuning parameters are also determined through cross validation. At the end, we compared the performance of these three methods based on the out-of-sample errors.

### 3.1 Random Forest

```r
fit.rf <- train(classe ~ ., data=training, method="rf", trControl=control, ntree=300)
fit.rf
```

```
##Random Forest 
##
##13737 samples
##   52 predictor
##    5 classes: 'A', 'B', 'C', 'D', 'E' 
##
##No pre-processing
##Resampling: Cross-Validated (5 fold) 
##Summary of sample sizes: 10990, 10989, 10990, 10991, 10988 
##Resampling results across tuning parameters:
##
##  mtry  Accuracy   Kappa    
##   2    0.9904638  0.9879360
##  27    0.9898817  0.9872000
##  52    0.9834763  0.9790937
##
##Accuracy was used to select the optimal model using  the largest value.
##The final value used for the model was mtry = 2. 
```

The optimal value of mtry obtained from the cross validation is 2. Then we applied our fitted model to the validation set. The out-of-sample errors are reported. 

```r
> pred.rf <- predict(fit.rf, valid)
> confusionMatrix(pred.rf, valid$classe)
```

```
##Confusion Matrix and Statistics
##
##          Reference
##Prediction    A    B    C    D    E
##         A 1673    9    0    0    0
##         B    1 1125   14    0    0
##         C    0    5 1010   24    1
##         D    0    0    2  939    3
##         E    0    0    0    1 1078
##
##Overall Statistics
##                                          
##               Accuracy : 0.9898          
##                 95% CI : (0.9869, 0.9922)
##    No Information Rate : 0.2845          
##    P-Value [Acc > NIR] : < 2.2e-16       
##                                          
##                  Kappa : 0.9871          
## Mcnemar's Test P-Value : NA              
```

Accuracy = 0.9898, out-of-sample error = 0.0102.

### 3.2 Decision Tree

```r
rpart.grid <- expand.grid(.cp=seq(0.01,0.05,0.01)) 
fit.tree <- train(classe~., data=training, method="rpart",trControl=control,tuneGrid=rpart.grid)
pred.tree <- predict(fit.tree, valid)
confusionMatrix(pred.tree, valid$classe)
```

```
##Confusion Matrix and Statistics
##
##          Reference
##Prediction    A    B    C    D    E
##         A 1498  196   69  106   25
##         B   42  669   85   86   92
##         C   43  136  739  129  131
##         D   33   85   98  553   44
##         E   58   53   35   90  790
##
##Overall Statistics
##                                          
##               Accuracy : 0.722           
##                 95% CI : (0.7104, 0.7334)
##    No Information Rate : 0.2845          
##    P-Value [Acc > NIR] : < 2.2e-16       
##                                          
##                  Kappa : 0.6467          
## Mcnemar's Test P-Value : < 2.2e-16       
```

Accurary = 0.722, out-of-sample error = 0.278.

### 3.3 GBM

```r
fit.gbm <- train(classe ~., method="gbm", data=training, trControl=control, verbose=F)
fit.gbm
```

```
##Stochastic Gradient Boosting 
##13737 samples
##   52 predictor
##    5 classes: 'A', 'B', 'C', 'D', 'E' 
##
##  interaction.depth  n.trees  Accuracy   Kappa    
##  1                   50      0.7530035  0.6869478
##  1                  100      0.8256543  0.7793491
##  1                  150      0.8541174  0.8153851
##  2                   50      0.8579759  0.8199875
##  2                  100      0.9083495  0.8839819
##  2                  150      0.9336827  0.9160723
##  3                   50      0.8956839  0.8679213
##  3                  100      0.9413268  0.9257482
##  3                  150      0.9611996  0.9509051
##Tuning parameter 'shrinkage' was held constant at a value of 0.1
##Tuning parameter 'n.minobsinnode' was held constant at a value of 10
##Accuracy was used to select the optimal model using  the largest value.
##The final values used for the model were n.trees = 150, interaction.depth =
## 3, shrinkage = 0.1 and n.minobsinnode = 10. 
 ```
 
 ```r
pred.gbm <- predict(fit.gbm, valid)
confusionMatrix(pred.gbm, valid$classe)
```

```
##          Reference
##Prediction    A    B    C    D    E
##         A 1644   44    0    1    2
##         B   17 1059   34    2   16
##         C    5   33  971   34    5
##         D    8    3   19  916   20
##         E    0    0    2   11 1039
##                                         
##               Accuracy : 0.9565         
##                 95% CI : (0.951, 0.9616)
##    No Information Rate : 0.2845         
##    P-Value [Acc > NIR] : < 2.2e-16      
##                                         
##                 Kappa : 0.945          
## Mcnemar's Test P-Value : 4.518e-07      

```

Accurary = 0.9565, out-of-sample error = 0.0435.

Conclusion: Since Random Forest has the lowest out-of-sample error, we prefer using Random Forest in prediction on the test data set.

## 4. Prediction on Test Data Set
Now we applied Random Forest to the testing data set to obtain the predictions. The result is shown below:

```r
pred.test <- predict(fit.rf, testing)
pred.test
##[1] B A B A A E D B A A B C B A E E A B B B
```
