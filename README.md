
<div style="background-color:#f5f5f5; padding: 20px; text-align: center; border-radius: 10px; margin-bottom: 20px;">

<h1 style="margin-bottom: 0;">
ISBS Online Symposium
</h1>
<p style="font-size: 1.2em; color: #555; margin-top: 5px;">
Functional Data Analysis for Sports Biomechanics
</p>

<div style="display: flex; justify-content: center; align-items: center; gap: 40px; margin-top: 15px;">

    <img src="isbs-logo.png" alt="ISBS Logo" width="120">
    <img src="fda-logo.png" alt="FDA Logo" width="120">

</div>

</div>

## Introduction

Welcome to this workshop on Functional Data Analysis (FDA) for Sports
Biomechanics, as part of the ISBS Online Symposium. This document serves
as a guide for working with time-series data and preparing it for
functional data analysis in R.

## Reading Time-Series Data

In this workshop, we will use the GaitRec dataset, specifically the
right leg vertical ground reaction force (vGRF) data. The data are
stored in a comma separated value (CSV) file on the [GaitRec Figshare
link](https://figshare.com/articles/dataset/GRF_F_V_RAW_right/11394825?backTo=%2Fcollections%2FGaitRec_A_large-scale_ground_reaction_force_dataset_of_healthy_and_impaired_gait%2F4788012&file=22063200)
and can be downloaded directly
[here](https://figshare.com/ndownloader/files/22063200). Each time
series is stored as a row of the CSV file, with associated metadata
including subject ID, session number, and trial ID. Since trials have
different durations, some rows contain missing values (NAs) where the
recorded data length varies. Our goal is to preprocess these data,
preparing them for functional data analysis.

### Step 1: Read in the csv file:

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
time_1 <- system.time(GRF_data_1 <- read.csv(file = "data/GRF_F_V_RAW_right.csv"))
time_2 <- system.time(GRF_data_2 <- readr::read_csv(file = "data/GRF_F_V_RAW_right.csv", 
                                                    col_types = c(rep("i", 3), rep("n", 405))))
time_3 <- system.time(GRF_data_3 <- data.table::fread(file = "data/GRF_F_V_RAW_right.csv"))
print(paste0("read.csv = ", round(time_1["elapsed"], 2), " seconds; read_csv = ", round(time_2["elapsed"], 2), " seconds; fread = ", round(time_3["elapsed"], 2), " seconds"))
```

------------------------------------------------------------------------

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

## Data Preprocessing

Discussion on normalization, interpolation, and smoothing techniques.

## Data Export

## Functional Data Analysis Basics

Overview of mean functions, variance functions, and functional PCA.

## Example Code

``` r
# Placeholder for example analysis
plot(cars)
```

![](README_files/figure-gfm/example-1.png)<!-- -->

## Conclusion

Final thoughts and next steps.
