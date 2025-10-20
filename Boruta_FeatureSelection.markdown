```r
# Install and explore the Boruta package:
install.packages("Boruta")
library(Boruta)
set.seed(1)
borutaFeatureSelectionMethod <- Boruta(diagnosis~.,data=trainingDataDT)
print(borutaFeatureSelectionMethod)
plot(borutaFeatureSelectionMethod, las=2)
```