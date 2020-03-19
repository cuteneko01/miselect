<!-- Badges -->
[![CRAN
Version](https://img.shields.io/cran/v/miselect?style=flat-square&color=blue&label=CRAN)](https://cran.r-project.org/package=miselect)
[![GitHub
Release](https://img.shields.io/github/v/release/umich-cphds/miselect?include_prereleases&label=Github&style=flat-square&color=blue)](https://github.com/umich-cphds/miselect)
[![Travis
CI](https://img.shields.io/travis/umich-cphds/miselect?style=flat-square)](https://travis-ci.org/umich-cphds/miselect)

Variable Selection for Multiply Imputed Data
============================================

Penalized regression methods, such as lasso and elastic net, are used in
many biomedical applications when simultaneous regression coefficient
estimation and variable selection is desired. However, missing data
complicates the implementation of these methods, particularly when
missingness is handled using multiple imputation. Applying a variable
selection algorithm on each imputed dataset will likely lead to
different sets of selected predictors, making it difficult to ascertain
a final active set without resorting to ad hoc combination rules.
'miselect' presents Stacked Adaptive Elastic Net (saenet) and Grouped
Adaptive LASSO (galasso) for continuous and binary outcomes. They, by
construction, force selection of the same variables across multiply
imputed data. 'miselect' also provides cross validated variants of these
methods.

Installation
------------

`miselect` can installed from Github via

    # install.packages("devtools")
    devtools::install_github("umich-cphds/miselect", build_opts = c())

The Github version may contain bug fixes not yet present on CRAN, so if
you are experiencing issues, you may want to try the Github version of
the package.

Example
-------

Here is how to use cross validated `saenet`. A nice feature of `saenet`
is that you can cross validate over `alpha` without having to use
`foldid`.

    library(miselect)
    library(mice)
    #> 
    #> Attaching package: 'mice'
    #> The following objects are masked from 'package:base':
    #> 
    #>     cbind, rbind

    set.seed(48109)
    # Using the mice defaults for sake of example only.
    mids <- mice(miselect.df, m = 5, printFlag = FALSE)
    dfs <- lapply(1:5, function(i) complete(mids, action = i))

    # Generate list of imputed design matrices and imputed responses
    x <- list()
    y <- list()
    for (i in 1:5) {
        x[[i]] <- as.matrix(dfs[[i]][, paste0("X", 1:20)])
        y[[i]] <- dfs[[i]]$Y
    }

    # Calculate observational weights
    weights  <- 1 - rowMeans(is.na(miselect.df))
    pf       <- rep(1, 20)
    adWeight <- rep(1, 20)
    alpha    <- c(.5 , 1)

    # Since 'Y' is a binary variable, we use 'family = "binomial"'
    fit <- cv.saenet(x, y, pf, adWeight, weights, family = "binomial",
                     alpha = alpha)

    # By default 'coef' returns the betas for (lambda.min , alpha.min)
    coef(fit)
    #> (Intercept)          X1          X2          X3          X4          X5 
    #>  0.07500679  1.48805119  0.57645656  0.00000000  1.79085524  0.05315903 
    #>          X6          X7          X8          X9         X10         X11 
    #>  0.00000000  1.78466568  0.00000000 -0.01937788  0.00000000  1.29382032 
    #>         X12         X13         X14         X15         X16         X17 
    #>  0.00000000  0.00000000  0.00000000 -0.14915055  0.00000000  0.28699265 
    #>         X18         X19         X20 
    #>  0.00000000 -0.23440273  0.00000000

You can supply different values of `lambda` and `alpha`. Here we use the
`lambda` and `alpha` selected by the one standard error rule

    coef(fit, lambda = fit$lambda.1se, alpha = fit$alpha.1se)
    #> (Intercept)          X1          X2          X3          X4          X5 
    #>   0.1180812   0.0000000   0.0000000   0.0000000   0.0000000   0.0000000 
    #>          X6          X7          X8          X9         X10         X11 
    #>   0.0000000   0.0000000   0.0000000   0.0000000   0.0000000   0.0000000 
    #>         X12         X13         X14         X15         X16         X17 
    #>   0.0000000   0.0000000   0.0000000   0.0000000   0.0000000   0.0000000 
    #>         X18         X19         X20 
    #>   0.0000000   0.0000000   0.0000000

Bugs
----

If you encounter a bug, please open an issue on the
[Issues](https://github.com/umich-cphds/miselect/issues) tab on Github
or send us an email.

Contact
-------

For questions or feedback, please email Jiacong Du at
<jiacong@umich.edu> or Alexander Rix <alexrix@umich.edu>.

References
----------

Variable selection with multiply-imputed datasets: choosing between
stacked and grouped methods. Jiacong Du, Jonathan Boss, Peisong Han,
Lauren J Beesley, Stephen A Goutman, Stuart Batterman, Eva L Feldman,
and Bhramar Mukherjee. 2020. arXiv:2003.07398
