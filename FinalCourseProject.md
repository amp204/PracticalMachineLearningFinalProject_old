Practical Machine Learning Final Project
Load required packages
library(caret)
library(randomForest)
Download the data:
Download the training and testing data files from the website. Replacing the #DIV/0 and Blanks with NAs:

train_www <- “http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv”
test_www <- “http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv”
training <- read.csv(url(train_www), na.strings=c(“NA”,“#DIV/0!”,“”))
testing <- read.csv(url(test_www), na.strings=c(“NA”,“#DIV/0!”,“”))
Check the dimensions of the training data:

dim(training)
19622 160
And see what it looks like:

str(training)
I want to remove the columns with NAs since they won’t be helpful in prediction:

training_lessna<- training[ , colSums(is.na(training)) == 0]
This brings the columns down to 60:

dim(training_lessna)
19622 60
I also want to remove the first few columns since it is just reference data that won’t be used in predicting.

I am going to use the nearZeroVar function to remove Constant and almost constant predictors across samples as they will not be informative.

nzv <- nearZeroVar(training_lessna,saveMetrics=TRUE)
training_good <- training_lessna[,nzv$nzv==FALSE]
Now I will use cross-validation and split my training set into a training and testing set, with 80% going to training and 20% going to cross-validation.

inTrain<-createDataPartition(y=training_good$classe, p=0.8, list=FALSE)
train<-training_good[inTrain,]
test<-training_good[-inTrain,]
Modeling
Will not fit the model - choose a decision tree since they are easy to intrepret and have better performance in non-linear setting.

model1 <- train(classe ~., method = “rpart”, data = train)
predictions<- predict(model1, newdata=test)
confusionMatrix(predictions,test$classe)

Result
Confusion Matrix and Statistics Reference 
Overall Statistics

           Accuracy : 0.9965         
             95% CI : (0.9945, 0.998)
No Information Rate : 0.2845         
P-Value [Acc > NIR] : < 2.2e-16      

              Kappa : 0.9956         
Mcnemar’s Test P-Value : NA

Therefore our out of sample accuracy is 99%.

I then apply this to the separated testing data to get final predictions:

predictions2 <- predict(model1, testing, type = “class”)
