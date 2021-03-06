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

Here are examples of STRIDE estimators when the nonparametric estimators of the survival distribution for different subgroups do and do  not adjust for landmarking and/or covariates.


```r
library(stride)

####################################################
##                                                ##
## EXAMPLE 1: STRIDE estimators do not adjust for ##
## landmarking or covariates.                     ##
##                                                ##
####################################################


# Setup parameters to generate the data
set.seed(1)  ## set the random seed generator.
censoring.rate <- 40  ## set the random censoring rate as 40%
p <- 2  ## number of population subgroups
n <- 2000  ## total sample size
m <- 4  ## number of mixture proportions.

simu.setting <- "HD-No-Covariates"  ## simulate the distribution functions similar to the Huntington disease study in Garcia and Parast (2020) where the distribution functions do NOT depend on covariates. The GenerateData() function still generates covariates, but the simulated distribution functions do not depend on them.

## compute the finite set of mixture proportions. We assume the mixture proportions are computed externally.
qvs <- qvs.values(p,m)

## generate the data
data.gen <- GenerateData(n,p,m,qvs,censoring.rate,simu.setting)
x <- data.gen$x  ## possibly censored survival outcomes
delta <- data.gen$delta  ## censoring indicator, 1= event time observed, 0= event time censored
q <- data.gen$q  ## mixture proportion for each individual
ww <- NULL ## no covariates
zz <- NULL ## no covariates


## Estimation procedures to run to estimate F(t|t0,z,w)
update.qs <- FALSE ## We do not update the mixture proportions q's. Updating mixture proportions is still in a trial phase.
run.NPMLEs <- TRUE  ## We run the type I nonparametric maximum likelihood estimator. 

##run.OLS <- TRUE #  If TRUE, then the output includes the estimated distribution function computed using an ordinary least squares influence function that adjusts for censoring using inverse probability weighting (IPW), augmented inverse probability weighting (AIPW), and imputation (IMP). See details in Wang et al (2012). 
run.OLS <- FALSE  

##run.WLS <- TRUE #  If TRUE, then the output includes the estimated distribution function computed using a weighted least squares influence function that adjusts for censoring using inverse probability weighting (IPW), augmented inverse probability weighting (AIPW), and imputation (IMP). See details in Wang et al (2012). 
run.WLS <- FALSE

##run.EFF <- TRUE # If TRUE, then the output includes the estimated distribution function computed using the efficient influence function based on Hilbert space projection theory that adjusts for censoring using inverse probability weighting (IPW), augmented inverse probability weighting (AIPW), and imputation (IMP). See details in Wang et al (2012). 
run.EFF <- FALSE

##run.EMPAVA <- TRUE # If TRUE, we compute the distribution function for the mixture data based on an expectation-maximization (EM) algorithm that uses the pool adjacent violators algorithm (PAVA) from isotone regression to yield a non-negative and monotone estimator. See details in Qing et al (2014)
run.EMPAVA <- FALSE

run.NPNA <- FALSE
run.NPNA_avg <- FALSE
run.NPNA_wrong <- FALSE


## The distribution function we are estimating is F(t|t0,z,w). 
tval <- seq(0,80,by=5)  ## tval refers to "t" in F(t|t0,z,w) 
tval0 <- 0 ##tval0 refers to "t0" in F(t|t0,z,w)
z.use <- NULL  ## z.use refers to "z" in  F(t|t0,z,w)
w.use <- NULL  ## w.use refers to "w" in F(t|t0,z,w)

## Setup to compute AUC/BS as in Garcia and Parast (2020). Only for simulated data.
## We compute the prediction accuracy measures, including the area under the receiver operating characteristic curve (AUC) and the Brier Score (BS). Prediction accuracy is only valid in simulation studies where know.true.groups=TRUE and true.group.identifier is available.
run.prediction.accuracy <- FALSE
do_cross_validation_AUC_BS <- FALSE
know.true.groups <- FALSE
true.group.identifier <- NULL


## Perform the estimation			
estimators.out <- stride.estimator(n,m,p,qvs,q,
				x,delta,ww,zz,
				run.NPMLEs,
				run.NPNA,
				run.NPNA_avg,
				run.NPNA_wrong,
				run.OLS,
        run.WLS,
        run.EFF,
				run.EMPAVA,
				tval,tval0,
				z.use,w.use,
				update.qs,
				know.true.groups,
				true.group.identifier,
				run.prediction.accuracy,
				do_cross_validation_AUC_BS)

####################################################
##                                                ##
## EXAMPLE 2: STRIDE estimators do  adjust for    ##
## landmarking or covariates.                     ##
##                                                ##
####################################################


# Setup parameters to generate the data
set.seed(1)  ## set the random seed generator.
censoring.rate <- 40  ## set the random censoring rate as 40%
p <- 2  ## number of population subgroups
n <- 2000  ## total sample size
m <- 4  ## number of mixture proportions.

simu.setting <- "HD-With-Covariates"  ## simulate the distribution functions similar to the Huntington disease study in Garcia and Parast (2020) where the distribution functions depend on covariates.

## compute the finite set of mixture proportions. We assume the mixture proportions are computed externally.
qvs <- qvs.values(p,m)

## generate the data
data.gen <- GenerateData(n,p,m,qvs,censoring.rate,simu.setting)
x <- data.gen$x  ## possibly censored survival outcomes
delta <- data.gen$delta  ## censoring indicator, 1= event time observed, 0= event time censored
q <- data.gen$q  ## mixture proportion for each individual
ww <- data.gen$ww  ## observed w-covariate for each individual
zz <- data.gen$zz  ## observed z-covariate for each individual


## Estimation procedures to run to estimate F(t|t0,z,w)
update.qs <- FALSE ## We do not update the mixture proportions q's. Updating mixture proportions is still in a trial phase.
run.NPMLEs <- TRUE  ## We run the type I nonparametric maximum likelihood estimator. 
run.NPNA <- TRUE  ## If TRUE, then the output includes the
#' estimated distribution function for mixture data that accounts for covariates and dynamic
#' landmarking. This estimator is called "NPNA" in Garcia and Parast (2020).
run.NPNA_avg <- FALSE
run.NPNA_wrong <- FALSE
run.OLS <- FALSE
run.WLS <- FALSE
run.EFF <- FALSE
run.EMPAVA <- FALSE


## The distribution function we are estimating is F(t|t0,z,w). 
tval <- seq(0,80,by=5)  ## tval refers to "t" in F(t|t0,z,w) 
tval0 <- c(0,20,30,40,50) ##tval0 refers to "t0" in F(t|t0,z,w)
z.use <- c(0,1)  ## z.use refers to "z" in  F(t|t0,z,w)
w.use <- seq(35,55,by=1)  ## w.use refers to "w" in F(t|t0,z,w)

## Setup to compute AUC/BS as in Garcia and Parast (2020). Only for simulated data.
## We compute the prediction accuracy measures, including the area under the receiver operating characteristic curve (AUC) and the Brier Score (BS). Prediction accuracy is only valid in simulation studies where know.true.groups=TRUE and true.group.identifier is available.
run.prediction.accuracy <- TRUE  
do_cross_validation_AUC_BS <- FALSE
know.true.groups <- TRUE
true.group.identifier <- data.gen$true.group.identifier


## Perform the estimation			
estimators.out <- stride.estimator(n,m,p,qvs,q,
				x,delta,ww,zz,
				run.NPMLEs,
				run.NPNA,
				run.NPNA_avg,
				run.NPNA_wrong,
				run.OLS,
        run.WLS,
        run.EFF,
				run.EMPAVA,
				tval,tval0,
				z.use,w.use,
				update.qs,
				know.true.groups,
				true.group.identifier,
				run.prediction.accuracy,
				do_cross_validation_AUC_BS)
```
