Predictive Modeling Analysis
================
Ming Wan, Victor Yuan

-   [Step 0: Load Packages and Data](#step-0-load-packages-and-data)
-   [Step 1: Unsupervised clustering:](#step-1-unsupervised-clustering)
    -   [PCA on training data:](#pca-on-training-data)
    -   [PCA projection of loadings to test data:](#pca-projection-of-loadings-to-test-data)
-   [Step 2: Supervised classification:](#step-2-supervised-classification)
    -   [logistic regression with elastic net regularization](#logistic-regression-with-elastic-net-regularization)

Step 0: Load Packages and Data
==============================

Load required packages:

``` r
#source("https://bioconductor.org/biocLite.R")
#biocLite('e1071')                                    # required for glmnet in caret
#biocLite('pROC')
library(pROC)
```

    ## Type 'citation("pROC")' for a citation.

    ## 
    ## Attaching package: 'pROC'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     cov, smooth, var

``` r
library(ggplot2)
library(limma)
library(caret)
```

    ## Loading required package: lattice

``` r
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
#library(glmnet)
```

Read in pre-processed data: \*Make sure the pre-processed data (data.txt, which is in data.zip) is present in the ../processed\_data/ directory.

``` r
setwd('../')                                           # note: all of these relative file path calls work only for knitting

# load data (pre-processed training set)
train.data <- read.table('../Data/Processed Data/data.txt')
str(train.data)
```

    ## 'data.frame':    464923 obs. of  45 variables:
    ##  $ PM104: num  0.9219 0.569 0.0435 0.0635 0.0368 ...
    ##  $ PM112: num  0.8546 0.6886 0.0724 0.0856 0.0364 ...
    ##  $ PM114: num  0.8319 0.6578 0.0906 0.0909 0.0393 ...
    ##  $ PM115: num  0.9021 0.6146 0.0665 0.0839 0.0389 ...
    ##  $ PM119: num  0.896 0.6565 0.0591 0.0987 0.0336 ...
    ##  $ PM120: num  0.8658 0.6338 0.0982 0.1254 0.0421 ...
    ##  $ PM123: num  0.8769 0.6214 0.0571 0.0582 0.0477 ...
    ##  $ PM124: num  0.8481 0.6759 0.0711 0.1022 0.0437 ...
    ##  $ PM130: num  0.821 0.6326 0.1116 0.1038 0.0449 ...
    ##  $ PM136: num  0.9024 0.569 0.0674 0.0671 0.0493 ...
    ##  $ PM139: num  0.8259 0.6098 0.0556 0.0628 0.0371 ...
    ##  $ PM142: num  0.8099 0.6414 0.1094 0.1451 0.0435 ...
    ##  $ PM153: num  0.8911 0.6525 0.0939 0.0888 0.044 ...
    ##  $ PM155: num  0.8384 0.5506 0.0802 0.0984 0.0458 ...
    ##  $ PM158: num  0.835 0.6604 0.1136 0.1122 0.0393 ...
    ##  $ PM167: num  0.8618 0.5825 0.0807 0.0774 0.0424 ...
    ##  $ PM181: num  0.9024 0.6273 0.0885 0.1101 0.0416 ...
    ##  $ PM20 : num  0.8925 0.677 0.1326 0.1507 0.0538 ...
    ##  $ PM205: num  0.851 0.641 0.139 0.142 0.125 ...
    ##  $ PM226: num  0.8822 0.693 0.0972 0.1074 0.0367 ...
    ##  $ PM227: num  0.9049 0.6118 0.0462 0.0719 0.0343 ...
    ##  $ PM233: num  0.8022 0.6165 0.0744 0.1352 0.0448 ...
    ##  $ PM243: num  0.8551 0.6229 0.0896 0.1001 0.0376 ...
    ##  $ PM249: num  0.9153 0.6164 0.0654 0.0946 0.0345 ...
    ##  $ PM29 : num  0.9114 0.6393 0.0399 0.0364 0.0296 ...
    ##  $ PM30 : num  0.8835 0.5735 0.0885 0.1435 0.0407 ...
    ##  $ PM4  : num  0.8463 0.5852 0.0733 0.0869 0.0408 ...
    ##  $ PM40 : num  0.8385 0.6786 0.0586 0.0718 0.0364 ...
    ##  $ PM41 : num  0.8149 0.6576 0.0663 0.1045 0.0485 ...
    ##  $ PM44 : num  0.8089 0.5596 0.1227 0.1471 0.0378 ...
    ##  $ PM46 : num  0.9026 0.6467 0.0751 0.1048 0.0372 ...
    ##  $ PM47 : num  0.8491 0.6345 0.0653 0.0967 0.0468 ...
    ##  $ PM52 : num  0.8891 0.5681 0.0782 0.1018 0.0362 ...
    ##  $ PM53 : num  0.8566 0.6803 0.0866 0.1076 0.0466 ...
    ##  $ PM54 : num  0.8513 0.7054 0.0998 0.1685 0.0427 ...
    ##  $ PM55 : num  0.8317 0.5979 0.0954 0.1153 0.0354 ...
    ##  $ PM58 : num  0.8322 0.6706 0.0754 0.1237 0.0492 ...
    ##  $ PM66 : num  0.9138 0.6296 0.0859 0.1162 0.041 ...
    ##  $ PM71 : num  0.7972 0.5837 0.1344 0.1746 0.0477 ...
    ##  $ PM72 : num  0.8706 0.6164 0.1114 0.1194 0.0396 ...
    ##  $ PM74 : num  0.8611 0.5987 0.0988 0.0997 0.037 ...
    ##  $ PM76 : num  0.8401 0.5985 0.0728 0.1062 0.0409 ...
    ##  $ PM84 : num  0.8955 0.6762 0.2148 0.1548 0.0433 ...
    ##  $ PM9  : num  0.8784 0.6027 0.1118 0.1305 0.0441 ...
    ##  $ PM98 : num  0.8279 0.629 0.0827 0.0837 0.0418 ...

``` r
## row names are CpG sites, column names are sample names

# load metadata
design <- read.csv("../Data/Processed Data/des.txt", sep="\t", header=TRUE)
str(design)
```

    ## 'data.frame':    45 obs. of  5 variables:
    ##  $ Samplename  : Factor w/ 45 levels "PM104","PM112",..: 1 2 3 4 5 6 7 8 9 10 ...
    ##  $ Sample_Group: Factor w/ 3 levels "CONTROL","IUGR",..: 1 1 1 3 3 1 2 1 2 1 ...
    ##  $ ga          : num  40.7 38.9 38.6 41.1 37.1 38 35.7 40 36.9 38.6 ...
    ##  $ sex         : Factor w/ 2 levels "F","M": 2 1 2 2 2 1 1 2 2 1 ...
    ##  $ Ethnicity   : Factor w/ 2 levels "Asian","Caucasian": 2 1 2 2 1 2 2 2 2 2 ...

``` r
colnames(train.data) == design$Samplename               # check that the samples are in same order
```

    ##  [1] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [15] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [29] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
    ## [43] TRUE TRUE TRUE

Step 1: Unsupervised clustering:
================================

---------------this section is exploratory analysis, we should move this to Nivi's exploratory file---------- As Rob suggested, PCA should be the precursor to supervised classification, more like an exploration.

PCA on training data:
---------------------

``` r
pc.train <- prcomp(t(scale(t(train.data), center = T, scale = T)), center = F, scale = F)

# look at the eigenvalues
plot(pc.train) # can we add some labels here (PC1, PC2, PC3,...)
```

![](PredictiveModeling_files/figure-markdown_github/pca-1.png) Say something about the plot

``` r
# first 2 PCS
PC123 <- data.frame(pc.train$rotation[,c("PC1","PC2", "PC3")])              # Take out first 3 PCs
PC123 <- PC123 %>% tibble::rownames_to_column('Samplename') %>%             # Put sample names into column to 
                    left_join(design, 'Samplename')                         # Join the metadata info 
```

    ## Warning in left_join_impl(x, y, by$x, by$y, suffix$x, suffix$y): joining
    ## factor and character vector, coercing into character vector

``` r
head(PC123)            
```

    ##   Samplename         PC1         PC2         PC3 Sample_Group   ga sex
    ## 1      PM104 -0.18077677 -0.45326948  0.17568389      CONTROL 40.7   M
    ## 2      PM112 -0.01134793 -0.12731742  0.02686157      CONTROL 38.9   F
    ## 3      PM114  0.12546312  0.10832086  0.01710225      CONTROL 38.6   M
    ## 4      PM115 -0.02530175  0.08189423  0.25572824        LOPET 41.1   M
    ## 5      PM119 -0.11471056 -0.05119294  0.07704112        LOPET 37.1   M
    ## 6      PM120  0.17288398  0.09457592 -0.17899820      CONTROL 38.0   F
    ##   Ethnicity
    ## 1 Caucasian
    ## 2     Asian
    ## 3 Caucasian
    ## 4 Caucasian
    ## 5     Asian
    ## 6 Caucasian

``` r
ggplot(PC123, aes(x = PC1, y = PC2)) + 
  geom_point(aes(color = Ethnicity)) +
  ggtitle('PC1 and PC2: Ethnicity')
```

![](PredictiveModeling_files/figure-markdown_github/Plot%20PC1%20PC2%20PC3-1.png)

``` r
ggplot(PC123, aes(x = PC1, y = PC3)) + 
  geom_point(aes(color = Ethnicity)) +
  ggtitle('PC1 and PC3: Ethnicity')
```

![](PredictiveModeling_files/figure-markdown_github/Plot%20PC1%20PC2%20PC3-2.png)

``` r
ggplot(PC123, aes(x = PC2, y = PC3)) + 
  geom_point(aes(color = Ethnicity)) +
  ggtitle('PC2 and PC3: Ethnicity')
```

![](PredictiveModeling_files/figure-markdown_github/Plot%20PC1%20PC2%20PC3-3.png) We can see from plotting the first three principal components that our groups (Asian, Caucasian) do not seem to separate. This indicates that the main drivers of the variance in the data is something else.

``` r
ggplot(PC123, aes(x = PC1, y = PC2)) + 
  geom_point(aes(color = sex)) +
  ggtitle('Sex')
```

![](PredictiveModeling_files/figure-markdown_github/Plot%20other%20metadata-1.png)

``` r
ggplot(PC123, aes(x = PC1, y = PC2)) + 
  geom_point(aes(color = ga)) +
  ggtitle('Gestational Age')
```

![](PredictiveModeling_files/figure-markdown_github/Plot%20other%20metadata-2.png)

``` r
ggplot(PC123, aes(x = PC1, y = PC2)) + 
  geom_point(aes(color = Sample_Group)) +
  ggtitle('Sample Group')
```

![](PredictiveModeling_files/figure-markdown_github/Plot%20other%20metadata-3.png) It's not clear that our other variables are driving the variance in the data (sex, gestational age, and sample group).

``` r
# scatter plot matrix for the first 5 PCs
splom(pc.train$rotation[,1:5], panel = panel.smoothScatter, raster = TRUE)
```

    ## (loaded the KernSmooth namespace)

![](PredictiveModeling_files/figure-markdown_github/Scatter%20plot%20matrix%20first%205%20PCs-1.png) What does this plot show?

PCA projection of loadings to test data:
----------------------------------------

``` r
# read pre-processed test data

# project PC loadings to test data
```

--------End of exploratory Analysis-----------

Step 2: Supervised classification:
==================================

logistic regression with elastic net regularization
---------------------------------------------------

``` r
#renamed just so that I can copy Amit's code

x.train <- train.data 
y.train <- design$Ethnicity 
```

``` r
k = 5
M = 3

fitControl <- trainControl(method = "repeatedcv", 
                                                     number = k,                 # Number of folds
                                                     repeats = M,
                                                     ## Estimate class probabilities
                                                      classProbs = TRUE,
                           ## Evaluate performance using 
                           ## the following function
                           summaryFunction = twoClassSummary,
                                                     savePredictions = TRUE      # Saves ROC results
                                                     )  
```

``` r
set.seed(2017)                                         # training models requires the use of random #s. Setting (set.seed()) the randomness                                                             ensures reproducibility

system.time(netFit <- train(x = t(x.train),   # samples need to be in rows, features need to be columns
                                y = y.train,                  
                                method = "glmnet",                     # glmnet model
                                trControl = fitControl,                # use fitControl to specify cross validation
                                preProcess = c( "center", "scale"),    # Center and Scale the data
                                metric = 'ROC')                        # ROC because distribution is slightly skewed
)
```

    ## Loading required package: glmnet

    ## Loading required package: Matrix

    ## Loading required package: foreach

    ## Loaded glmnet 2.0-5

    ## 
    ## Attaching package: 'glmnet'

    ## The following object is masked from 'package:pROC':
    ## 
    ##     auc

    ##    user  system elapsed 
    ## 1628.49   83.61 1744.03

``` r
netFit
```

    ## glmnet 
    ## 
    ##     45 samples
    ## 464923 predictors
    ##      2 classes: 'Asian', 'Caucasian' 
    ## 
    ## Pre-processing: centered (464923), scaled (464923) 
    ## Resampling: Cross-Validated (5 fold, repeated 3 times) 
    ## Summary of sample sizes: 37, 36, 35, 36, 36, 36, ... 
    ## Resampling results across tuning parameters:
    ## 
    ##   alpha  lambda      ROC        Sens       Spec     
    ##   0.10   0.02429921  0.9968254  0.8555556  1.0000000
    ##   0.10   0.07684085  0.9968254  0.8555556  1.0000000
    ##   0.10   0.24299210  0.9968254  0.8111111  1.0000000
    ##   0.55   0.02429921  1.0000000  0.9111111  1.0000000
    ##   0.55   0.07684085  1.0000000  0.8666667  1.0000000
    ##   0.55   0.24299210  1.0000000  0.7444444  1.0000000
    ##   1.00   0.02429921  0.9746032  0.8666667  0.9904762
    ##   1.00   0.07684085  0.9793651  0.8444444  0.9904762
    ##   1.00   0.24299210  0.9624339  0.3888889  1.0000000
    ## 
    ## ROC was used to select the optimal model using  the largest value.
    ## The final values used for the model were alpha = 0.55 and lambda
    ##  = 0.2429921.

Cross validation with a fold of k = 5 (making each fold 9 samples large), was used to determine the optimal tuning parameters.

Horvath et al. (2013) uses an 'elastic net generalized linear model' to build an across-tissue DNAm predictor on age. Since our data is the same type, we'll try glmnet.

Horvath, S. (2013). DNA methylation age of human tissues and cell types. Genome Biology, 14(10), R115. <http://doi.org/10.1186/gb-2013-14-10-r115>

``` r
trellis.par.set(caretTheme())
ggplot(netFit)
```

    ## Warning: Ignoring unknown aesthetics: shape

![](PredictiveModeling_files/figure-markdown_github/examine%20results-1.png)

``` r
#heatmap of results
plot(netFit, metric = "ROC", plotType = "level",
     scales = list(x = list(rot = 90)))
```

![](PredictiveModeling_files/figure-markdown_github/examine%20results-2.png)

``` r
predictors <- predictors(netFit)
predictors
```

    ##  [1] "cg06903451" "cg09843049" "cg07835437" "cg12983196" "cg06615678"
    ##  [6] "cg15486123" "cg10265016" "cg07547054" "cg13921903" "cg00574513"
    ## [11] "cg08704934" "cg01770232" "cg26295559" "cg00812861" "cg19878200"
    ## [16] "cg14581129" "cg05393297" "cg25025879" "cg16329197" "cg12011926"
    ## [21] "cg16808927" "cg08100221" "cg05795554" "cg04287289" "cg26513180"
    ## [26] "cg12602405" "cg02159489" "cg06595479" "cg07035552"

``` r
length(predictors) 
```

    ## [1] 29

Looks like our model has chosen 'r length(predictors)' CpGs that can be used to predict ethnicity.

``` r
glmImp <- varImp(netFit, scale = F) # gives the t-statistic for all CpGs in the dataset
plot(glmImp, top = 35)
```

![](PredictiveModeling_files/figure-markdown_github/plot%20top%2035-1.png)

``` r
# this is adapted from Amit's code from lec 19
#use repeated CV to estimate test performance
set.seed(2018)

# list of lists containing fold ids
folds <- lapply(1:M, function(i) createFolds(y.train, k = k))

netTesterror <- lapply(folds, function(i){
  lapply(i, function(j){
    # tune parameters with CV
    set.seed(2019)
    fitControl <- trainControl(method = "repeatedcv", 
                                                     number = k,                 
                                                     repeats = M,
                                                     classProbs = TRUE,
                           summaryFunction = twoClassSummary,
                                                     savePredictions = TRUE)
    
    
    # build elastic net classifier
    netFit <- train(x =  t(x.train)[-j,],   
                                y = y.train[-j],                  
                                method = "glmnet",                     
                                trControl = fitControl,
                                preProcess = c( 'center', 'scale'),
                                metric = 'ROC')   
    
    # Estimate probabilities of test predictions
    probTest <- predict(netFit, t(x.train)[j,], type = 'prob')
    ethProb <- probTest[,'Asian']
    ethProb

  })
})
```

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

    ## Warning in lognet(x, is.sparse, ix, jx, y, weights, offset, alpha, nobs, :
    ## one multinomial or binomial class has fewer than 8 observations; dangerous
    ## ground

``` r
netTesterror
```

    ## [[1]]
    ## [[1]]$Fold1
    ## [1] 0.80118476 0.02555646 0.06140903 0.03609474 0.63628429 0.06697999
    ## [7] 0.02904903 0.01430149 0.02977636
    ## 
    ## [[1]]$Fold2
    ## [1] 0.01567475 0.09577317 0.10272328 0.02675467 0.22820570 0.06872002
    ## [7] 0.02847293 0.78315843 0.45184197
    ## 
    ## [[1]]$Fold3
    ## [1] 0.61757208 0.07147420 0.11283338 0.07895275 0.03222931 0.03186154
    ## [7] 0.21784797 0.04108041 0.74462068
    ## 
    ## [[1]]$Fold4
    ## [1] 0.11329844 0.03583564 0.02853341 0.01188968 0.04921686 0.57152557
    ## [7] 0.55362247 0.04194895
    ## 
    ## [[1]]$Fold5
    ##  [1] 0.02965134 0.03002370 0.49793858 0.04697648 0.15374738 0.72156837
    ##  [7] 0.34235511 0.08516250 0.04360862 0.04466175
    ## 
    ## 
    ## [[2]]
    ## [[2]]$Fold1
    ##  [1] 0.09926479 0.22277651 0.09937258 0.66416574 0.09863577 0.19545900
    ##  [7] 0.56935626 0.15586287 0.10741511 0.49424112
    ## 
    ## [[2]]$Fold2
    ## [1] 0.02448076 0.86914086 0.03124585 0.03216389 0.01960078 0.55173553
    ## [7] 0.07994581 0.11865110 0.15303932
    ## 
    ## [[2]]$Fold3
    ##  [1] 0.03015257 0.02538668 0.48226619 0.04076482 0.02797293 0.02124027
    ##  [7] 0.70731037 0.70048656 0.02074951 0.03318546
    ## 
    ## [[2]]$Fold4
    ## [1] 0.55275105 0.05372028 0.08534017 0.01485792 0.50606052 0.07385628
    ## [7] 0.04835082 0.01602613
    ## 
    ## [[2]]$Fold5
    ## [1] 0.04123112 0.04458234 0.14975392 0.34973409 0.07974504 0.03352487
    ## [7] 0.73904092 0.04817553
    ## 
    ## 
    ## [[3]]
    ## [[3]]$Fold1
    ## [1] 0.06029396 0.09745268 0.13621062 0.11473110 0.58727326 0.06781633
    ## [7] 0.82735669 0.05010994
    ## 
    ## [[3]]$Fold2
    ##  [1] 0.03119275 0.02360133 0.03230596 0.04316843 0.02804458 0.12959037
    ##  [7] 0.02778506 0.04456812 0.62451216 0.44215394
    ## 
    ## [[3]]$Fold3
    ## [1] 0.01993918 0.48605734 0.03706410 0.01695667 0.38662658 0.06139780
    ## [7] 0.10972265 0.20097334
    ## 
    ## [[3]]$Fold4
    ## [1] 0.76314865 0.12058900 0.13613982 0.10654128 0.06640907 0.69094074
    ## [7] 0.11308927 0.15194954 0.30339688
    ## 
    ## [[3]]$Fold5
    ##  [1] 0.67549439 0.03971941 0.16015573 0.02865107 0.63336816 0.71353931
    ##  [7] 0.07854770 0.04623719 0.04903272 0.04775205

``` r
# Computer classification performance measures
# enet
Performance <- mapply(function(x, y){
  auc <- pROC::roc(y.train[unlist(x)], unlist(y),
                   direction ='<',
                   levels = c('Caucasian', 'Asian'),
                   percent = TRUE)
  list(tpr = auc$sensitivities,
       fpr = 100 - auc$specificities,
       auc = round(auc$auc, 2))
}, x = folds, y = netTesterror)
Performance
```

    ##     [,1]       [,2]       [,3]      
    ## tpr Numeric,46 Numeric,46 Numeric,46
    ## fpr Numeric,46 Numeric,46 Numeric,46
    ## auc 100        100        98.48

``` r
# plot ROC curve

plot(Performance['tpr',][[1]] ~ Performance['fpr',][[1]],
     type = 'l', col = 1, xlab = '100 - sensitivity',
     ylab = 'Sensitivity', main = 'Enet')
for(i in length(folds)){
  points(Performance['tpr',][[i]] ~ Performance['fpr',][[i]],
         type = 'l', col = 2)
}
text(x = 60, y = 40, labels =
       paste0('mean AUC = ', round(mean(unlist(Performance['auc',])), 1),
              '+/-', round(sd(unlist(Performance['auc',])), 1), '%'))
```

![](PredictiveModeling_files/figure-markdown_github/ROC%20curve-1.png)
