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
