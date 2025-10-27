```{R}
# Machine Learning with caret Package & Synthetic Data Generation in R

# This script demonstrates machine learning using the `caret` package on the Breast Cancer Wisconsin dataset and introduces synthetic 
# data generation with SMOTE.

# Load required library: caret
# caret is a comprehensive package for machine learning in R
library("caret")

# Download the Breast Cancer Wisconsin dataset from Kaggle and import into R
# Dataset URL: https://www.kaggle.com/datasets/uciml/breast-cancer-wisconsin-data
# Save the file as 'data.csv' in your working directory or provide full path

# Import the dataset
# data <- read.csv("/Users/Anthony/masters-ulster/DataScience/data.csv")  # Replace "data.csv" with actual file path if needed

# Select all rows and the first 32 columns (includes id, diagnosis, and 30 features)
# Column 1: id, Column 2: diagnosis (M= malignant, B= benign), Columns 3:32: features
data <- data[, c(1:32)]

# Create a 70/30 train-test split using stratified sampling on 'diagnosis'
# Ensures both training and test sets have similar proportion of classes
partition <- createDataPartition(data$diagnosis, p = 0.7, list = FALSE)

trainingData <- data[partition, ]   # 70% for training
testData     <- data[-partition, ]  # 30% for testing

# Learn more about createDataPartition:
# ?createDataPartition
# Documentation: https://topepo.github.io/caret/data-splitting.html

# Build a K-Nearest Neighbors (KNN) model using caret
# Formula: diagnosis ~ .  → Use all other columns as predictors
model <- train(diagnosis ~ ., method = "knn", data = trainingData)

# Display the model results
# Shows accuracy and kappa for k = 5, 7, 9 (default tuning grid)
model

# Make predictions on the test set
testingTheModel <- predict(model, newdata = testData)

# Check the data type of the true labels (reference)
class(testData$diagnosis)

# Check the data type of the predictions (data)
class(testingTheModel)

testData$diagnosis <- factor(testData$diagnosis, levels = levels(testingTheModel))

# Generate confusion matrix to evaluate performance
# Includes Accuracy, Kappa, Sensitivity, Specificity specificity, etc.
confusionMatrix(testingTheModel, testData$diagnosis)

# Check class distribution in test set to compute No Information Rate (NIR)
# NIR = proportion of majority class (baseline accuracy if predicting majority only)
table(testData$diagnosis)

# Example: If output is B:107, M:63 → NIR = 107/170 ≈ 62.94%

# Perform a one-sample proportion test to compare model accuracy vs. NIR
# Replace 116 and 107 with actual correct predictions and NIR count if different
prop.test(c(116, 107), c(170, 170))

# Set up 10-fold cross-validation (repeated once by default)
# method = "repeatedcv", number = 10 → 10 folds
# savePredictions = TRUE → Store predictions for analysis
control <- trainControl(method = "repeatedcv", number = 10, savePredictions = TRUE)

# Re-train KNN model with 10-fold CV
model <- train(diagnosis ~ ., method = "knn", data = trainingData, trControl = control)
model

# View cross-validation predictions
cv <- model$pred
head(cv)

# Check number of observations per fold
# Training size (~399) / 10 ≈ ~40 per fold
table(model$pred$Resample)

# List all available models in caret
# Shows short names for use in method argument
names(getModelInfo())

# Train a Support Vector Machine with linear kernel
model_svm <- train(diagnosis ~ ., method = "svmLinear", data = trainingData, trControl = control)
model_svm

# View SVM cross-validation results
cv_svm <- model_svm$pred
table(cv_svm$Resample)

# Optional Exercise:
# Train multiple models (e.g., knn, svmLinear, rpart, rf) and compare performance
# Extract Accuracy, Sensitivity, Specificity and plot using ggplot2 or base R

# ==============================================================================
# SYNTHETIC DATA GENERATION USING SMOTE
# ==============================================================================

# Install and load smotefamily package for SMOTE
# install.packages("smotefamily")  # Uncomment if not installed
install.packages("smotefamily")
library(smotefamily)

# View SMOTE documentation
?SMOTE

# Generate a sample imbalanced dataset
# 1000 samples, 2 features, 1 label ('result'), 70% majority class
dataSet <- sample_generator(1000, ratio = 0.70)

# Check class distribution
table(dataSet$result)
# Expected: ~700 majority (p), ~300 minority (n)

# Apply SMOTE to balance classes
# K=5: Use 5 nearest neighbors to generate synthetic samples
generateData <- SMOTE(dataSet[, c(1, 2)], dataSet[, c(3)], K = 5)

# Combine original + synthetic data
newDataSet <- generateData$data

# Check new class balance
table(newDataSet$class)
```

```{r}
```
