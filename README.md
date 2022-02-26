# IntegratedLearner - Integrated machine learning for multi-omics prediction and classification 

The repository houses the **`IntegratedLearner`** source code for multi-omics classification and prediction in both cross-sectional and longitudinal datasets while allowing adjustment for multiple covariates and repeated measures. Both binary and continuous outcomes (univariate) are supported. 

**`IntegratedLearner`** provides an integrated machine learning framework to 1) consolidate predictions by borrowing information across several longitudinal and cross-sectional omics data layers and 2) decipher the mechanistic role of individual biological entities (e.g., taxa, genes, metabolites, etc.) that can potentially lead to new sets of testable hypotheses and experimental validation studies.

**`IntegratedLearner`** algorithm proceeds by 1) fitting a machine learning algorithm per-layer to predict outcome (```base_learner```) and 2) combining the layer-wise cross-validated predictions using a meta model (```meta_learner```) to generate final predictions based on all available data points. As a default choice, we recommend Bayesian additive regression trees (BART) as the base learner (```base_learner='SL.BART'```) and non-negative least squares/ rank loss minimization as the meta model algorithm (```meta_learner='SL.nnls.auc'```). ```'SL.nnls.auc'``` fits a non-negative least squares (in case of continuous outcome) and rank loss minimization (in case of binary outcome) on layer-wise cross-validated predictions to generate the final predictions and quantify per-layer contribution in the final predictions.

The following libraries need to be included for the R code to run:

```r
library(caret)
library(tidyverse)
library(SuperLearner)
library(glmnetUtils)
library(devtools)
devtools::source_url("https://github.com/himelmallick/IntegratedLearner/blob/master/scripts/IntegratedLearner_CV.R?raw=TRUE") 
```

## Instructions for use

### IntegratedLearner main function

#### Usage

```
run_integrated_learner_CV(feature_table, sample_metadata, feature_metadata, ...)
```

#### Arguments

* ```feature_table ```: Data frame representing concatenated multi-omics features with features in rows (```rownames```) and samples in columns (```colnames```).
* ```sample_metadata ```: Data frame of sample-specific metadata. Must have a column named ```subjectID``` describing per-subject unique identifiers. For longitudinal designs, this variable is expected to have non-unique values. Additionally, a column named ```Y``` must be present which is the outcome of interest (can be binary or continuous). Row names of ```sample_metadata``` must match the column names of ```feature_table```.
* ```feature_metadata ```: Data frame containing feature-specific metadata. Must have a column named ```featureID``` describing per-feature unique identifiers. Additionally, if multiple omics layers are present, a column named ```featureType``` should describe the corresponding source layer (e.g. metagenomics, metabolomics, etc.). Row names must match that of ```feature_table```.
* ```feature_metadata_valid ```: Optional feature table from validation set. Must have the exact same structure as `feature_table`. 
* ```sample_metadata_valid```: Optional sample-specific metadata table from independent validation set. Must have the exact same structure as `sample_metadata`. 
* ```family```: A character string representing one of the built-in families. Currently, ```gaussian()``` and ```binomial()``` are supported.
* ```folds```: Integer. Number of folds for cross-validation. Default is 5.
* ```base_learner ```: Character string representing the name of the ```SL``` base-learner in stacked generalization and optionally for joint learner (see example). Check out the [SL user manual](https://cran.r-project.org/web/packages/SuperLearner/vignettes/Guide-to-SuperLearner.html) for all available options. Default is ```'SL.BART'```
* ```meta_learner```: Character string representing the name of the ```SL``` meta-learner in stacked generalization (see example). Check out the [SL user manual](https://cran.r-project.org/web/packages/SuperLearner/vignettes/Guide-to-SuperLearner.html) for all available options. Default is ```'SL.nnls.auc'```
* ```run_concat```: Logical value representing whether a joint (concatenated) model should also be run (see tutorial). Default is TRUE.
* ```run_stacked```: Logical value representing whether a stacked model should also be run (see tutorial). Default is TRUE.
* ```print_learner```: Logical value representing whether a summary of fit should be printed. Default is TRUE.
* ```verbose```: Logical value for printing progress during the computation (helpful for debugging). Default is FALSE. 
* ```...```: Additional arguments for `SL` tuning parameters.

#### The IntegratedLearner workflow
![Flow Chart](/images/Flowchart.png)

#### Value

* ```SL_fits```: A list of ```SL``` prediction results from all individual base learners, the meta learner, and optionally the joint (concatenation) learner.
* ```model_fits```: A list of ```base_learner``` objects extracted from ```SL_fits``` for all individual base learners, meta learner, and optionally the joint (concatenation) learner.
* ```X_train_layers```: Input feature matrices for individual layers for training data.
* ```Y_train```: Input response vector for training data.
* ```yhat.train```: Predictions for training data from all individual base learners, the meta learner, and optionally the joint (concatenation) learner.
* ```X_test_layers```: Input feature matrices for individual layers for test data. Available if ```feature_table_valid``` is provided. 
* ```Y_test```: Input response vector for test data.
* ```weights```: Estimated layer weights in the meta model. Available if ```run_stacked=TRUE``` and ```meta_learner='SL.nnls.auc'```.
* ```AUC.train```/```R2.train```: AUC/ R2 metrics calculated on training data using ```yhat.train``` and ```Y_train```.
* ```AUC.test```/```R2.test```: AUC/ R2 metrics calculated on test data using ```yhat.test``` and ```Y_test```.
* ```...```: Additional arguments containing information about inputs. 

## Getting Started
The package vignette demonstrates how to use the **IntegratedLearner** workflow to perform a multi-omics prediction and classification task. This vignette can be viewed online [here](http://htmlpreview.github.io/?https://github.com/himelmallick/IntegratedLearner/blob/master/vignettes/IntegratedLearner.html).


Citation
--------

If you use `IntegratedLearner` in your work, please cite the following:

Mallick H et al. (2021+). An Integrated Bayesian Framework for Multi-omics Prediction and Classification (In Submission).

Issues
------

We are happy to troubleshoot any issues with the package. Please contact the maintainer via email or [open an issue](https://github.com/himelmallick/IntegratedLearner/issues) in the GitHub repository.
