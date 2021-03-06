First, please note that this package is in a very rough state. That being said, it is still usable for someone with a little bioinformatics/programming/R experience.

To install in R:

```
install.packages('devtools', dep=T)
library(devtools)
install_github('cMonkeyNwInf','djreiss',subdir='cMonkeyNwInf')
```

Here are notes on Running the package (copied from an email):

The package will also require packages 'lars', 'glmnet', 'Matrix', 'multicore' and all of their dependencies.

Load the RData file and look at the 'env.nwInf' function. 

Look at 'colMap', 'envMap' and 'predictors' for examples. 

1. 'predictors' is the list of TFs (a character vector)
2. 'colMap' gives the experimental metadata such as time-series, time point, etc.
3. 'envMap' gives the environmental data such as oxygen level, etc. These are used in addition to the 'predictors' as potential influences.

The 'predictors' list is a list of TFs. They need to have the names that are in the rownames of the ratios matrix -- i.e., same format, etc.

The 'envMap' is a matrix of environmental data as a function of experiment. It should have the same format as the expression ratios matrix, and the same number of columns (and same column names -- experiment names). The rows from this matrix will be included as possible predictors along with the TFs listed in the 'predictors' list.

The 'colMap' is probably the most confusing. Here is a summary:

It is a data.frame, with 1 row for each condition in the microarray data. If a condition is part of a time series, then it will have

```
$ isTs : logi TRUE (True if this condition is part of a Time series)
$ is1stLast: Factor w/ 4 levels "e","f","m","l": 3 (it is "m" for middle of a time series; the other three options are: "e"=equilibrium/not-Time-series, "f"=first-in-Ts and "l"=last-in-Ts)
$ prevCol : chr "T0_N0025_r3" (the previous time-point/condition-name was "T0_N0025_r3")
$ delta.t : int 24 (between the previous and current time-points 24 minutes have passed)
$ time : int (time point in minutes)
$ condName : Named chr "T24_N0000_r3" (this condition name is "T24_N0000_r3", thus this is the name of condition 200)
$ ts.ind : NOT USED, just place an integer in there
$ numTS : not used, just place an integer in there
```

the other option is that a condition is not part of a time series, it will look like this:

```
$ isTs : logi FALSE (False because this condition is not part of a time series)
$ is1stLast: Factor w/ 4 levels "e","f","m","l": 1 (it is "e" for equilibrium)
$ prevCol : logi NA (it has no previous conditions/column, thus we set it to NA)
$ delta.t : logi NA (del.t is not defined as it is not part of a time series, thus we set it to NA)
$ time : int NA (time point in minutes set to NA)
$ condName : Named chr "dinI___U_N0025_r1" (this is the name of condition 1)
$ ts.ind : NOT USED, just place an integer in there
$ numTS : not used, just place an integer in there
```

Assuming you have all the pieces in place (tfs list ('predictors'), 'colMap', and [optionally] 'envMap') set up in your R environment, 

Then, the following in R should work (ignore the "halo" in the name of the function):

```
library( cMonkeyNwInf ) ## load the package

e.coeffs <- runnit.wrapper.halo( e, cv.choose="min+4se", tf.groups=999, alpha=0.8, n.boot=1, tau=10,
                                  r.cutoff=Inf, r.filter=Inf, weighted=T, aic.filter=Inf, plot=F )
```

(Don't mind the 'halo' in the name of this function - that is historical only.)
This will take a while (30 minutes - 2 hours depending on your computer).
It will run in parallel using all cores on your computer. If you get an error, setting the variable

```
DEBUG = TRUE
```

in your R environment will turn off parallelization so that you can use

```
traceback()
```

to see what went wrong (and then email me or open up an issue).
