---
output: github_document
---

<!-- README.md is generated from README.Rmd. Please edit that file -->


# stride

<!-- badges: start -->
This R package implements nonparametric estimation of survival models for mixture data.
Mixture data is data collected from multiple populations of interest, but we do not know from which population each observation belongs.  The corresponding references are:

Garcia, T.P. and Parast, L. (2020). Dynamic landmark prediction for mixture data. Biostatistics,  doi:10.1093/biostatistics/kxz052.

Garcia, T.P., Marder, K. and Wang, Y. (2017). Statistical modeling of Huntington disease onset.
In Handbook of Clinical Neurology, vol 144, 3rd Series, editors Andrew Feigin and Karen E. Anderson.

Qing, J., Garcia, T.P., Ma, Y., Tang, M.X., Marder, K., and Wang, Y. (2014).
Combining isotonic regression and EM algorithm to predict genetic risk under monotonicity constraint.
Annals of Applied Statistics, 8(2), 1182-1208.

Wang, Y., Garcia, T.P., and Ma. Y. (2012).  Nonparametric estimation for censored mixture data with
application to the Cooperative Huntington's Observational Research Trial. Journal of the American Statistical Association, 107, 1324-1338.
<!-- badges: end -->



## Installation

You can install the development version from [GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("tpgarcia/stride", force=TRUE)
```
## Example

This is a basic example which shows you how to solve a common problem:


```r
library(stride)

# Setup parameters to generate the data
set.seed(1)
censoring.rate <- 40
p <- 2
n <- 2000
m <- 4
tval <- seq(0,80,by=5)  
tval0 <- c(0,20,30,40,50)
z.use <- c(0,1)
w.use <- seq(35,55,by=1)
run.prediction.accuracy <- TRUE
update.qs <- FALSE
simu.setting <- "2A"
covariate.dependent <- TRUE
run.NPMLEs <- TRUE
run.NPNA <- TRUE
run.OLS <- FALSE
run.WLS <- FALSE
run.EFF <- FALSE
run.NPNA_avg <- FALSE
run.NPNA_wrong <- FALSE
do_cross_validation_AUC_BS <- FALSE
know.true.groups <- TRUE


## compute the finite set of mixture proportions
qvs <- qvs.values(p,m)

## generate the data

data.gen <- GenerateData(n,p,m,qvs,censoring.rate,simu.setting,covariate.dependent)

x <- data.gen$x
delta <- data.gen$delta
q <- data.gen$q
ww <- data.gen$ww
zz <- data.gen$zz


## true group membership (needed to compute the AUC/BS for simulated data)
true.groups <- data.gen$true.groups

## Perform the estimation			
estimators.out <- stride.estimator(n,m,p,qvs,q,
				x,delta,ww,zz,
				run.NPMLEs,
				run.NPNA,
				run.NPNA_avg,
				run.NPNA_wrong,
				tval,tval0,
				z.use,w.use,
				update.qs,
				know.true.groups,
				true.groups,
				run.prediction.accuracy,
				do_cross_validation_AUC_BS)
```