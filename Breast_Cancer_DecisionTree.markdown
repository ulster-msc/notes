```r
# Install the following package:
install.packages("C50")

# Import the Breast Cancer Wisconsin (Diagnostic) dataset and name it ‘data’.

# Like before – lets shuffle the rows in the dataset to ensure representation. We also only select columns 2 to 32.
set.seed(1)
DataDF <- data[ sample(nrow(data), replace = FALSE),c(2:32)]

# We also need to cast the label as a factor/categorical variable for classification:
DataDF$diagnosis <- factor(DataDF$diagnosis)

# The dataset labels are imbalanced but let’s leave this as it is.
table(DataDF$diagnosis)

# Let’s create our training and test sets:
trainingDataDT <- DataDF[c(1:500),]
testDataDT <- DataDF[c(501:569),]

# Let’s build the decision tree:
library(C50)
decisionTree <- C5.0(diagnosis ~., data=trainingDataDT)
summary(decisionTree)

# You can also plot the tree:
plot(decisionTree)

# Test the model on the test set:
decisionTreePredictions <- predict(decisionTree, testDataDT)

# You can now display the results using the code:
table(decisionTreePredictions, testDataDT$diagnosis)

# or:
CrossTable(x = decisionTreePredictions, y = testDataDT$diagnosis, prop.chisq=FALSE)

# Now try improving the model using boosting via the trials argument.
decisionTree <- C5.0(diagnosis ~., data=trainingDataDT, trials = 10)
```