
<!-- README.md is generated from README.Rmd. Please edit that file -->

# imputeGeneric

<!-- badges: start -->

[![R-CMD-check](https://github.com/torockel/imputeGeneric/workflows/R-CMD-check/badge.svg)](https://github.com/torockel/imputeGeneric/actions)
<!-- badges: end -->

The goal of imputeGeneric is to ease the implementation of imputation
functions.

## Installation

You can install the development version of imputeGeneric from
[GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("torockel/imputeGeneric")
```

## Purpose

The aim of imputeGeneric is to make the implementation and usage of
imputation methods easier. The main function of the package is
`impute_iterative()`. This function can turn any
[parsnip](https://cran.r-project.org/package=parsnip) model into an
imputation method. Furthermore, other customized approaches can be used
in a general imputation framework. For more information, see the
documentations of `impute_iterative()`, `impute_supervised()`,
`impute_unsupervised()` and the following examples.

## Examples

### Simple example

The use of a parsnip model for imputation is demonstrated using
regression trees from the rpart package via parsnip
(`decision_tree("regression")`). First, a data set with missing values
is created. Then, this data set is imputed once with regression trees
using only completely observed rows and columns for the model building.

``` r
library(imputeGeneric)
library(parsnip)
# create data set
set.seed(123)
ds_mis <- data.frame(X = rnorm(100), Y = rnorm(100))
ds_mis$Z <- 5 + 2* ds_mis$X + ds_mis$Y + rnorm(100)
ds_mis$Z[sample.int(100, 30)] <- NA
ds_mis$Y[sample.int(100, 20)] <- NA
# impute data set
ds_imp <- impute_iterative(ds_mis, decision_tree("regression"), max_iter = 1)
anyNA(ds_imp)
#> [1] FALSE
```

To use other parsnip models instead of regression trees, only the
`model_spec_parsnip` argument must be altered. E.g. for linear
regression instead of regression trees use `linear_reg()`.

``` r
ds_imp_lm <- impute_iterative(ds_mis, linear_reg(), max_iter = 1)
anyNA(ds_imp_lm)
#> [1] FALSE
```

### More complex example

Many aspects of the imputation can be specified and customized. The
missing values can be initially imputed e.g.??with per column mean values
(`initial_imputation_fun = missMethods::impute_mean`). In addition, all
objects and columns can be used for the imputation models
(`rows_used_for_imputation = "all"` and
`cols_used_for_imputation = "all"`). Furthermore, the imputation can be
iterative. The iteration will be stopped, if either the difference
between two imputed data sets falls below a threshold
(`stop_fun = stop_ds_difference, stop_fun_args = list(eps = 0.1)`) or
the maximum number of iterations (`max_iter = 5`) is reached.

``` r
ds_imp2 <- impute_iterative(
  ds_mis, decision_tree("regression"), 
  initial_imputation_fun = missMethods::impute_mean,
  cols_used_for_imputation = "all",
  rows_used_for_imputation = "all",
  stop_fun = stop_ds_difference,
  stop_fun_args = list(eps = 0.1),
  max_iter = 5)
anyNA(ds_imp2)
#> [1] FALSE
```
