# Machine Learning Tell Me To Stay Or Leave
layout: post

### Background
I had ever been in a situation that quite a few of people were leaving the company. I was asked by people and myself that whether I would stay or leave? Well, I thought it is good to have sort of humor in my life. A job should be interesting even when I was facing some difficulties.

Then, it turned out to me to create a predictive model to predict my future using machine learning. R was used with the caret packages.

### Data
Using some known and guessed values, an employee dataset of the organization was created.


```r
suppressMessages(require(caret))
suppressMessages(require(randomForest))

data0 <- read.csv("emp.csv")[, c(-1,-2)] ## The 1st and 2nd fields contains identities and were removed.
me <- data0[2,]                          ## The 2nd row was myself and was removed.
data <- data0[-2,]
head(data)
```

```
##     age gender family work_time manager_leave manager_of_manager_leave
## 1 50-60      m      y       1~5             1                        1
## 3 40-50      f      y      5~10             0                        1
## 4 30-40      f      y       1~5             0                        1
## 5 30-40      f      y      5~10             0                        1
## 6 20-30      m      n    0~0.25             0                        1
## 7 20-30      m      n    0~0.25             0                        1
##   team_member_leave contractor grade status
## 1                 1          n    11 Stayed
## 3                 1          n     7 Stayed
## 4                 1          n     7 Stayed
## 5                 1          n     5 Stayed
## 6                 0          n     3   Left
## 7                 1          n     5 Stayed
```

```r
dim(data)
```

```
## [1] 26 10
```

There 10 fields which are straightforward. There 26 records and myself.

I then created the training and testing datasets. I didn't set a seed to reproduce the same results as I thought God may affects my life instead of determining. :)


```r
inTrain <- createDataPartition(data$status, p=0.6, list=FALSE)
training <- data[inTrain,]
testing <- data[-inTrain,]
```

### Create Random Forrest Model
Random Forrest came into my mind as the dataset was small. Default parameters were used.

```r
model_rf <- train( status ~ ., data = training, method = "rf")
predict_rf <- predict(model_rf, testing)
confusion_rf <- confusionMatrix(predict_rf, testing$status)
confusion_rf$overall[1]
```

```
##  Accuracy 
## 0.8888889
```
The accuracy of the model was 88.9% on the testing set when I ran it. 

### Predict My Future
I then used it to predict myself.

```r
my_future <- predict(model_rf, me)
my_future
```

```
## [1] Left
## Levels: Left Stayed
```
The random forrest model told me to leave. So was the model's prediction correct or not? :)

Have a nice weekend all!

