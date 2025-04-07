ISBS Online Symposium
================

<div style="text-align: center;">

<img src="isbs-logo.png" alt="ISBS Logo" width="200" style="margin-right: 50px;">
<img src="fda-logo.png" alt="FDA Logo" width="200">

</div>

## Introduction

Welcome to this workshop on Functional Data Analysis (FDA) for Sports
Biomechanics, as part of the ISBS Online Symposium. This document serves
as a guide for working with time-series data and preparing it for
functional data analysis in R.

- This tutorial <u>**does**</u> cover, basic data import, formatting,
  preparation and inspecting techniques. It will teach you how to go
  from raw biomechanical time series’ to functional data that are ready
  to be analysed.

- This tutorial <u>**does not**</u> cover more advanced analytical
  techniques from FDA – we have [a
  book](https://link.springer.com/book/10.1007/978-3-031-68862-1) and
  [material from a full one-day
  course](https://github.com/edwardgunning/ISBS-Short-Course) for this.

- We encourage participants to ask questions and discuss how the
  material relates to their own work, either during the presentation by
  using the “raise hand” function ion zoom, in the chat, or during the
  dedicated question time at the end.

## 🖥 Computing Pre-requisites

### R and RStudio

If you want to follow along and program yourself, either retrospectively
or in real time, you should have the following software installed:

- **The R Language for Statistical Computing**
  - It can be downloaded from <https://cloud.r-project.org>
  - For further assistance see [this video by RStudio
    education](https://vimeo.com/203516510)
- **The RStudio Integrated Development Environment (IDE)**
  - It can be downloaded from <https://posit.co/>
  - For further assistance see [this video by RStudio
    education](https://vimeo.com/203516510) (**Note**: The RStudio
    company has changed to Posit PBC, so there may be some minor
    differences)

**Note**: If you are unable to install R and RStudio, you can work with
a free, lite web version of RStudio called [*posit
cloud*](https://posit.cloud/). Watch [this video from Posit
PBC](https://www.youtube.com/watch?v=-fzwm4ZhVQQ) to set up an account
and get started.

We also recommend setting up an RStudio project to work and store your
files for this workshop in – see [this helpful guide on setting up
projects by Posit
PBC](https://support.posit.co/hc/en-us/articles/200526207-Using-RStudio-Projects).

**IMPORTANT**: **We do not require any previous knowledge of R
programming or FDA**. We have structured the lecture and practical
sessions in such a way that all levels of experience will be catered
for. However, if interested, our favourite (free!) resources for getting
up to speed with R are:

- [R for Data Science (2nd Edition)](https://r4ds.hadley.nz/) by Hadley
  Wickham, Mine Çetinkaya-Rundel, and Garrett Grolemund.

- [R Programming for Data
  Science](https://bookdown.org/rdpeng/rprogdatascience/) by Roger D.
  Peng.

# ⏱️ Schedule

These times are in British Standard Time (BST):

|              Time | Topic                                |    Lead     |
|------------------:|:-------------------------------------|:-----------:|
| $12.00$ - $12.25$ | Welcome, Introduction and Background |     DH      |
| $12.25$ - $13.30$ | Practical Tutorial                   |     EG      |
| $13.30$ - $14.00$ | Q&A and Discussion                   | DH, EG & JW |

------------------------------------------------------------------------

# Main Tutorial

### Reading Time-Series Data

In this workshop, we will use the GaitRec dataset, specifically the
right leg vertical ground reaction force (vGRF) data. The data are
stored in a comma separated value (CSV) file on the [GaitRec Figshare
link](https://figshare.com/articles/dataset/GRF_F_V_RAW_right/11394825?backTo=%2Fcollections%2FGaitRec_A_large-scale_ground_reaction_force_dataset_of_healthy_and_impaired_gait%2F4788012&file=22063200)
and can be downloaded directly
[here](https://figshare.com/ndownloader/files/22063200). Each time
series is stored as a row of the CSV file, with associated metadata
including subject ID, session number, and trial ID. Since trials have
different durations, some rows contain missing values (`NA`s) where the
recorded data length varies. Our goal is to preprocess these data,
preparing them for functional data analysis.

### Read in the csv file:

There are many options to read in text files in R (e.g., comma or tab
separated values files). Here, we’ll use base R `read.csv()` function.
Since we are working within a project, we just have to point to the
*relative path* to the file.

``` r
GRF_data <- read.csv(file = "data/GRF_F_V_RAW_right.csv")
```

------------------------------------------------------------------------

***Aside:***

There exist faster functions in specialized packages for reading in
large csv files. Let’s time trial `read.csv()` with `read_csv()` from
the `readr` package and `fread` from the `data.table` package. From just
one run it seems `fread()` is marginally faster than `read_csv()`, but
both are orders of magnitude faster than `read.csv()`.

``` r
runtime_1 <- system.time(GRF_data_1 <- read.csv(file = "data/GRF_F_V_RAW_right.csv"))
runtime_2 <- system.time(GRF_data_2 <- readr::read_csv(file = "data/GRF_F_V_RAW_right.csv", 
                                                    col_types = c(rep("i", 3), rep("n", 405))))
runtime_3 <- system.time(GRF_data_3 <- data.table::fread(file = "data/GRF_F_V_RAW_right.csv"))

# Examine the results:
print(paste0("read.csv = ", round(runtime_1["elapsed"], 2), " seconds; read_csv = ",
             round(runtime_2["elapsed"], 2), " seconds; fread = ",
             round(runtime_3["elapsed"], 2), " seconds"))
rm(list = paste0("GRF_data_", 1:3)) # remove the created data objects from our environments
```

------------------------------------------------------------------------

### Inspecting the loaded data

The data is read in as a data frame, which is useful for storing tabular
data where the columns are heterogenous in nature (e.g., contains some
numeric and some factors or strings).

``` r
class(GRF_data)
```

    ## [1] "data.frame"

This dataset contains over $75,000$ rows. This meaans its a fantastic
resource for statistics and machine learning applications. However, for
the purpose of this tutorial, we’ll take a random sample of $200$ rows
to make it more manageable.

``` r
dim(GRF_data) # check dimensions
```

    ## [1] 75732   408

``` r
sample_inds <- sample(seq_len(length.out = nrow(GRF_data)), size = 200)
GRF_data <- GRF_data[sample_inds, ]
dim(GRF_data) # check dimensions again
```

    ## [1] 200 408

We can also split the data out into the first three columns (subject,
session and trial IDs) and the remaining $405$ columns which include the
sampled time series. Since the remaining $405$ columns are all numeric
containing time series values, it is appropriate to store them as a
matrix.

``` r
meta_df <- GRF_data[, 1:3] # first three columns
GRF_matrix <- as.matrix(GRF_data[, - c(1:3)])  # remaining columns
```

We can also create a $405$-dimensional vector representing the time
argument. Since these data are sampled at $250$ Hz, we have that each
time difference is $1/250$.

``` r
frames_per_second <- 250
seconds_per_frame <- 1 / frames_per_second
time_seq <- seq(0, seconds_per_frame * (405 - 1), by = seconds_per_frame)
```

We can use the `matplot()` function to plot the columns of a matrix, so
we need to transpose (rotate) `GRF_matrix` when passing it as an
argument using the `t()` function.

``` r
matplot(x = time_seq,
        y = t(GRF_matrix), 
        type = "b", 
        cex = 0.5, 
        pch = 20, 
        ylab = "vGRF",
        xlab = "time (seconds)")
```

![](README_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

## Data Preprocessing and Preparation

In this section, we’ll discuss a number of important issues in data
preparation.

For this, we’ll use the `fda` package so we need to load it.

``` r
library(fda)
```

### Issue 1: Smoothing

- The first issue is representing each sampled time series as a *smooth*
  function (or curve).

- In the`fda` package, this representation is done using a **basis
  function representation**.

- In short, we use a linear combination (or weighted sum) of some set of
  basis functions that *we know*, to approximate each individual curve.
  Then, under the hood, the data are stored as the combination of the
  basis (i.e., the known functions) and vector of basis coefficients for
  an individual curve (i.e., the weights).

- We can choose the basis coefficients based on whether we want to
  **smooth** or interpolate the raw data.

#### Demonstration: Representing a single curve

We’ll extract the first row of the dataset. We only take the non `NA`
values, and take the correponding values for the time argument.

``` r
GRF_obs_full <- GRF_matrix[1,]
GRF_obs <- GRF_obs_full[!is.na(GRF_obs_full)]
time_seq_obs <- time_seq[seq_len(length(GRF_obs))]
```

``` r
Bspline_basis_k50 <- create.bspline.basis(rangeval = range(time_seq_obs),
                                           nbasis = 50)

GRF_obs_1_fdSmooth <- smooth.basis(argvals = time_seq_obs,
                                   y = GRF_obs, 
                                   fdParobj = Bspline_basis_k50)

par(mfrow = c(1, 2))
plot(Bspline_basis_k50, knots = FALSE, lty = 1)
abline(h = 1, lwd = 1.5)
title("BSpline Basis")
plot.fd(fd(coef = diag(GRF_obs_1_fdSmooth$fd$coefs[,1]),
                  Bspline_basis_k50), lty = 1, ylim = c(0, 800))
```

    ## [1] "done"

``` r
points(x = time_seq_obs, GRF_obs, pch = 20, col = "grey")
lines.fdSmooth(GRF_obs_1_fdSmooth, col = "black", lwd = 1.5)
title("Weighted BSpline Basis to Represent Curve")
```

![](README_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

**What is happening under the hood?**

$\rightarrow$ We have gone from a vector of discrete values to a
representation of the functional data in terms of a vector of basis
coefficients and a set of basis functions.

These are stored as an `fd` object:

``` r
GRF_obs_1_fd <- GRF_obs_1_fdSmooth$fd
GRF_obs_1_fd[["coefs"]]
```

    ##               [,1]
    ## bspl4.1   24.08241
    ## bspl4.2   69.23778
    ## bspl4.3   96.64736
    ## bspl4.4  198.27381
    ## bspl4.5  340.99138
    ## bspl4.6  447.81026
    ## bspl4.7  534.91860
    ## bspl4.8  615.08891
    ## bspl4.9  687.25999
    ## bspl4.10 745.77887
    ## bspl4.11 793.72978
    ## bspl4.12 796.17414
    ## bspl4.13 779.71989
    ## bspl4.14 755.70246
    ## bspl4.15 723.52903
    ## bspl4.16 700.55688
    ## bspl4.17 680.60705
    ## bspl4.18 670.10102
    ## bspl4.19 675.41740
    ## bspl4.20 680.89286
    ## bspl4.21 676.02143
    ## bspl4.22 677.39813
    ## bspl4.23 676.32673
    ## bspl4.24 679.07617
    ## bspl4.25 684.35169
    ## bspl4.26 683.26125
    ## bspl4.27 684.23444
    ## bspl4.28 685.46798
    ## bspl4.29 682.25840
    ## bspl4.30 680.96510
    ## bspl4.31 695.95729
    ## bspl4.32 717.06143
    ## bspl4.33 742.98979
    ## bspl4.34 766.78154
    ## bspl4.35 786.40931
    ## bspl4.36 797.28332
    ## bspl4.37 797.49978
    ## bspl4.38 784.23969
    ## bspl4.39 764.88535
    ## bspl4.40 730.51958
    ## bspl4.41 678.08551
    ## bspl4.42 604.32404
    ## bspl4.43 520.26815
    ## bspl4.44 423.11783
    ## bspl4.45 310.88290
    ## bspl4.46 194.67288
    ## bspl4.47 107.56882
    ## bspl4.48  46.73550
    ## bspl4.49  28.14312
    ## bspl4.50  25.10916

``` r
GRF_obs_1_fd[["basis"]]
```

    ## $call
    ## basisfd(type = type, rangeval = rangeval, nbasis = nbasis, params = params, 
    ##     dropind = dropind, quadvals = quadvals, values = values, 
    ##     basisvalues = basisvalues)
    ## 
    ## $type
    ## [1] "bspline"
    ## 
    ## $rangeval
    ## [1] 0.000 0.848
    ## 
    ## $nbasis
    ## [1] 50
    ## 
    ## $params
    ##  [1] 0.01804255 0.03608511 0.05412766 0.07217021 0.09021277 0.10825532
    ##  [7] 0.12629787 0.14434043 0.16238298 0.18042553 0.19846809 0.21651064
    ## [13] 0.23455319 0.25259574 0.27063830 0.28868085 0.30672340 0.32476596
    ## [19] 0.34280851 0.36085106 0.37889362 0.39693617 0.41497872 0.43302128
    ## [25] 0.45106383 0.46910638 0.48714894 0.50519149 0.52323404 0.54127660
    ## [31] 0.55931915 0.57736170 0.59540426 0.61344681 0.63148936 0.64953191
    ## [37] 0.66757447 0.68561702 0.70365957 0.72170213 0.73974468 0.75778723
    ## [43] 0.77582979 0.79387234 0.81191489 0.82995745
    ## 
    ## $dropind
    ## NULL
    ## 
    ## $quadvals
    ## NULL
    ## 
    ## $values
    ## list()
    ## 
    ## $basisvalues
    ## list()
    ## 
    ## $names
    ##  [1] "bspl4.1"  "bspl4.2"  "bspl4.3"  "bspl4.4"  "bspl4.5"  "bspl4.6" 
    ##  [7] "bspl4.7"  "bspl4.8"  "bspl4.9"  "bspl4.10" "bspl4.11" "bspl4.12"
    ## [13] "bspl4.13" "bspl4.14" "bspl4.15" "bspl4.16" "bspl4.17" "bspl4.18"
    ## [19] "bspl4.19" "bspl4.20" "bspl4.21" "bspl4.22" "bspl4.23" "bspl4.24"
    ## [25] "bspl4.25" "bspl4.26" "bspl4.27" "bspl4.28" "bspl4.29" "bspl4.30"
    ## [31] "bspl4.31" "bspl4.32" "bspl4.33" "bspl4.34" "bspl4.35" "bspl4.36"
    ## [37] "bspl4.37" "bspl4.38" "bspl4.39" "bspl4.40" "bspl4.41" "bspl4.42"
    ## [43] "bspl4.43" "bspl4.44" "bspl4.45" "bspl4.46" "bspl4.47" "bspl4.48"
    ## [49] "bspl4.49" "bspl4.50"
    ## 
    ## attr(,"class")
    ## [1] "basisfd"

### Issue 2: Time Normalization

- The standard functional data analysis software (i.e., the `fda`
  package) requires the functional data to be represented on a common
  domain (**Note:** There are some exceptions such as the work of Gellar
  et al. (2014) and Sangalli et al. (2010) do not require a common
  domain/ equal length curves).

- However, in practice many biomechanical time series are of different
  lengths because people take
  dhttps://link.springer.com/book/10.1007/978-3-031-68862-1ifferent
  amounts of time to complete a movement.

- 

#### 2 (a) <u>Common Approach</u>: Resample to Time Normalize and then Smooth

#### 2 (b) <u>More Principled Approach</u>: Do Smoothing and Time Normalization Together

## Data Export

## Summarizing the Data (basics)

Overview of mean functions, variance functions, and functional PCA.

## Conclusion

Final thoughts and next steps.

## References

- Gellar, Jonathan E., Elizabeth Colantuoni, Dale M. Needham, and
  Ciprian M. Crainiceanu. “Variable-Domain Functional Regression for
  Modeling ICU Data.” Journal of the American Statistical Association
  109, no. 508 (2014): 1425–39.
  <https://doi.org/10.1080/01621459.2014.940044>.

- Sangalli, Laura M., Piercesare Secchi, Simone Vantini, and Valeria
  Vitelli. “K-Mean Alignment for Curve Clustering.” Computational
  Statistics & Data Analysis 54, no. 5 (2010): 1219–33.
  <https://doi.org/10.1016/j.csda.2009.12.008>.

- <https://github.com/gsimchoni/mocap> `mocap` R package.
