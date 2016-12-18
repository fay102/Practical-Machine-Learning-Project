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
Data cleaning: we first remove data with large portion (more than 70%) of missing values, then pick up the predictors which are related to the response variable.
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
We applied Random Forest and Decision Tree to fit the training data set. For Random Forest, a 5-fold cross validation is used to determine the optimal value of mtry. For Decision Tree, we also used 5-fold cross validation to choose the best complexity parameter. 
### 3.1 Random Forest
```r
> fit.rf <- train(classe ~ ., data=training, method="rf", trControl=control, ntree=300)
> fit.rf
Random Forest 

13737 samples
   52 predictor
    5 classes: 'A', 'B', 'C', 'D', 'E' 

No pre-processing
Resampling: Cross-Validated (5 fold) 
Summary of sample sizes: 10990, 10989, 10990, 10991, 10988 
Resampling results across tuning parameters:

  mtry  Accuracy   Kappa    
   2    0.9904638  0.9879360
  27    0.9898817  0.9872000
  52    0.9834763  0.9790937

Accuracy was used to select the optimal model using  the largest value.
The final value used for the model was mtry = 2. 
```
The optimal value of mtry obtained from the cross validation is 2. Then we applied our fitted model to the validation set. The out-of-sample accurary is reported.  
```r
> pred.rf <- predict(fit.rf, valid)
> confusionMatrix(pred.rf, valid$classe)
Confusion Matrix and Statistics

          Reference
Prediction    A    B    C    D    E
         A 1673    9    0    0    0
         B    1 1125   14    0    0
         C    0    5 1010   24    1
         D    0    0    2  939    3
         E    0    0    0    1 1078

Overall Statistics
                                          
               Accuracy : 0.9898          
                 95% CI : (0.9869, 0.9922)
    No Information Rate : 0.2845          
    P-Value [Acc > NIR] : < 2.2e-16       
                                          
                  Kappa : 0.9871          
 Mcnemar's Test P-Value : NA              

Statistics by Class:

                     Class: A Class: B Class: C Class: D Class: E
Sensitivity            0.9994   0.9877   0.9844   0.9741   0.9963
Specificity            0.9979   0.9968   0.9938   0.9990   0.9998
Pos Pred Value         0.9946   0.9868   0.9712   0.9947   0.9991
Neg Pred Value         0.9998   0.9970   0.9967   0.9949   0.9992
Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
Detection Rate         0.2843   0.1912   0.1716   0.1596   0.1832
Detection Prevalence   0.2858   0.1937   0.1767   0.1604   0.1833
Balanced Accuracy      0.9986   0.9923   0.9891   0.9865   0.9980
```
### 3.2 Decision Tree
```r
> rpart.grid <- expand.grid(.cp=seq(0.01,0.05,0.01)) 
> fit.tree <- train(classe~., data=training, method="rpart",trControl=control,tuneGrid=rpart.grid)
> 
> pred.tree <- predict(fit.tree, valid)
> confusionMatrix(pred.tree, valid$classe)
Confusion Matrix and Statistics

          Reference
Prediction    A    B    C    D    E
         A 1498  196   69  106   25
         B   42  669   85   86   92
         C   43  136  739  129  131
         D   33   85   98  553   44
         E   58   53   35   90  790

Overall Statistics
                                          
               Accuracy : 0.722           
                 95% CI : (0.7104, 0.7334)
    No Information Rate : 0.2845          
    P-Value [Acc > NIR] : < 2.2e-16       
                                          
                  Kappa : 0.6467          
 Mcnemar's Test P-Value : < 2.2e-16       

Statistics by Class:

                     Class: A Class: B Class: C Class: D Class: E
Sensitivity            0.8949   0.5874   0.7203  0.57365   0.7301
Specificity            0.9060   0.9357   0.9097  0.94717   0.9509
Pos Pred Value         0.7909   0.6869   0.6273  0.68020   0.7700
Neg Pred Value         0.9559   0.9043   0.9390  0.91897   0.9399
Prevalence             0.2845   0.1935   0.1743  0.16381   0.1839
Detection Rate         0.2545   0.1137   0.1256  0.09397   0.1342
Detection Prevalence   0.3218   0.1655   0.2002  0.13815   0.1743
Balanced Accuracy      0.9004   0.7615   0.8150  0.76041   0.8405
> fancyRpartPlot(fit.tree$finalModel)

```
## 4. Prediction on Test Data Set
