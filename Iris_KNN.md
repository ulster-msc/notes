```{r}
# Install the following packages:
install.packages("gmodels")
install.packages("class")

# Use the following code.

# This sets the seed to ensure reproducibility.
set.seed(1)

# We are creating a new data frame called IrisData which is based on the imported data frame called Iris. The following code also randomly shuffles the rows in the dataset which is important to gain representation. We are also removing column 1 as this is an ID column which we do not need. We are retaining columns 2 to 6.
IrisData <- Iris[ sample(nrow(Iris), replace = FALSE), c(2,3,4,5,6)]

# This code converts the label to a categorical variable for classification:
IrisData$Species <- factor(IrisData$Species)

# The following code will tell you the class distribution in the data:
table(IrisData$Species)
prop.table(table(IrisData$Species))

# The following code standardises the numeric columns (features/variables) using the scale function. This ensures that all features are on a similar distribution using standard units. You could also use normalisation.
IrisData$SepalLengthCm <- scale(IrisData$SepalLengthCm)
IrisData$SepalWidthCm <- scale(IrisData$SepalWidthCm)
IrisData$PetalLengthCm <- scale(IrisData$PetalLengthCm)
IrisData$PetalWidthCm <- scale(IrisData$PetalWidthCm)

# To verify this, the mean and SD of a standardised variable should be 0 (mean) and 1 (SD) respectively.
round(mean(IrisData$SepalLengthCm))
round(sd(IrisData$SepalLengthCm))

# This code creates training and test datasets.
trainingData <- IrisData[c(1:100),c(1,2,3,4)]
testData <- IrisData[c(101:150),c(1,2,3,4)]

# Notice that we leave out the labels for K-NN. However, we store the labels separately using the following code:
trainingLabels <- IrisData[c(1:100),c(5)]
testingLabels <- IrisData[c(101:150),c(5)]

# The following code builds our K-NN model where k=3:
library(class)
knnModelAndPredictedLabels <- knn(train = trainingData, test = testData, cl = trainingLabels, k=3)

# Have a look at the documentation for this function:
?knn

# Now present the results using cross tabulation:
# Basic table:
table(testingLabels, knnModelAndPredictedLabels)

# Or an alternative:
library(gmodels)
CrossTable(x = testingLabels, y = knnModelAndPredictedLabels, prop.chisq=FALSE)
```
