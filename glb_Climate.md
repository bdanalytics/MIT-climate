# Global Climate: Temp <regression/classification>
bdanalytics  

**  **    
**Date: (Sat) Mar 21, 2015**    

# Introduction:  

Data: 
Source: 
    Training:   https://courses.edx.org/c4x/MITx/15.071x_2/asset/climate_change.csv  
    New:        <prdct_url>  
Time period: 



# Synopsis:

Based on analysis utilizing <> techniques, <conclusion heading>:  

### ![](<filename>.png)

## Potential next steps include:

# Analysis: 

```r
rm(list=ls())
set.seed(12345)
options(stringsAsFactors=FALSE)
source("~/Dropbox/datascience/R/mydsutils.R")
source("~/Dropbox/datascience/R/myplot.R")
source("~/Dropbox/datascience/R/mypetrinet.R")
# Gather all package requirements here
suppressPackageStartupMessages(require(plyr))
suppressPackageStartupMessages(require(reshape2))

#require(sos); findFn("pinv", maxPages=2, sortby="MaxScore")

# Analysis control global variables
glb_is_separate_predict_dataset <- FALSE    # or TRUE
glb_predct_var <- "Temp"           # or NULL
glb_predct_var_name <- paste0(glb_predct_var, ".predict")
glb_id_vars <- c("Year", "Month")                # or NULL

glb_exclude_vars_as_features <- NULL                      
# List chrs converted into factors 
# glb_exclude_vars_as_features <- union(glb_exclude_vars_as_features, 
#                                       c("<col_name>")     # or NULL
#                                       )
# List feats that shd be excluded due to known causation by prediction variable
glb_exclude_vars_as_features <- union(glb_exclude_vars_as_features, 
                                      c("Year", "Month")     # or NULL
                                      )

glb_is_regression <- TRUE; glb_is_classification <- FALSE

glb_mdl <- glb_sel_mdl <- NULL
glb_models_df <- data.frame()

script_df <- data.frame(chunk_label="import_data", chunk_step_major=1, chunk_step_minor=0)
print(script_df)
```

```
##   chunk_label chunk_step_major chunk_step_minor
## 1 import_data                1                0
```

## Step `1`: import data

```r
glb_entity_df <- myimport_data(
    url="https://courses.edx.org/c4x/MITx/15.071x_2/asset/climate_change.csv", 
    comment="glb_entity_df", force_header=TRUE, print_diagn=TRUE)
```

```
## [1] "Reading file ./data/climate_change.csv..."
## [1] "dimensions of data in ./data/climate_change.csv: 308 rows x 11 cols"
##   Year Month   MEI    CO2     CH4     N2O  CFC.11  CFC.12      TSI
## 1 1983     5 2.556 345.96 1638.59 303.677 191.324 350.113 1366.102
## 2 1983     6 2.167 345.52 1633.71 303.746 192.057 351.848 1366.121
## 3 1983     7 1.741 344.15 1633.22 303.795 192.818 353.725 1366.285
## 4 1983     8 1.130 342.25 1631.35 303.839 193.602 355.633 1366.420
## 5 1983     9 0.428 340.17 1648.40 303.901 194.392 357.465 1366.234
## 6 1983    10 0.002 340.30 1663.79 303.970 195.171 359.174 1366.059
##   Aerosols  Temp
## 1   0.0863 0.109
## 2   0.0794 0.118
## 3   0.0731 0.137
## 4   0.0673 0.176
## 5   0.0619 0.149
## 6   0.0569 0.093
##     Year Month    MEI    CO2     CH4     N2O  CFC.11  CFC.12      TSI
## 51  1987     7  1.847 349.91 1681.24 306.138 234.778 422.707 1365.822
## 139 1994    11  1.278 357.56 1781.02 310.914 271.087 520.879 1365.829
## 223 2001    11 -0.180 369.69 1793.55 316.729 260.434 543.226 1366.520
## 233 2002     9  0.808 370.66 1780.92 317.015 258.269 543.078 1366.458
## 269 2005     9  0.255 376.66 1787.78 319.108 250.998 541.206 1365.799
## 271 2005    11 -0.407 378.29 1803.68 319.525 251.048 541.756 1365.779
##     Aerosols  Temp
## 51    0.0108 0.238
## 139   0.0159 0.248
## 223   0.0021 0.491
## 233   0.0022 0.413
## 269   0.0037 0.498
## 271   0.0037 0.478
##     Year Month    MEI    CO2     CH4     N2O  CFC.11  CFC.12      TSI
## 303 2008     7  0.003 386.42 1782.93 321.372 244.434 535.026 1365.672
## 304 2008     8 -0.266 384.15 1779.88 321.405 244.200 535.072 1365.657
## 305 2008     9 -0.643 383.09 1795.08 321.529 244.083 535.048 1365.665
## 306 2008    10 -0.780 382.99 1814.18 321.796 244.080 534.927 1365.676
## 307 2008    11 -0.621 384.13 1812.37 322.013 244.225 534.906 1365.707
## 308 2008    12 -0.666 385.56 1812.88 322.182 244.204 535.005 1365.693
##     Aerosols  Temp
## 303   0.0033 0.406
## 304   0.0036 0.407
## 305   0.0043 0.378
## 306   0.0046 0.440
## 307   0.0048 0.394
## 308   0.0046 0.330
## 'data.frame':	308 obs. of  11 variables:
##  $ Year    : int  1983 1983 1983 1983 1983 1983 1983 1983 1984 1984 ...
##  $ Month   : int  5 6 7 8 9 10 11 12 1 2 ...
##  $ MEI     : num  2.556 2.167 1.741 1.13 0.428 ...
##  $ CO2     : num  346 346 344 342 340 ...
##  $ CH4     : num  1639 1634 1633 1631 1648 ...
##  $ N2O     : num  304 304 304 304 304 ...
##  $ CFC.11  : num  191 192 193 194 194 ...
##  $ CFC.12  : num  350 352 354 356 357 ...
##  $ TSI     : num  1366 1366 1366 1366 1366 ...
##  $ Aerosols: num  0.0863 0.0794 0.0731 0.0673 0.0619 0.0569 0.0524 0.0486 0.0451 0.0416 ...
##  $ Temp    : num  0.109 0.118 0.137 0.176 0.149 0.093 0.232 0.078 0.089 0.013 ...
##  - attr(*, "comment")= chr "glb_entity_df"
## NULL
```

```r
if (glb_is_separate_predict_dataset) {
    glb_predct_df <- myimport_data(
        url="<prdct_url>", 
        comment="glb_predct_df", print_diagn=TRUE)
} else {
#     glb_predct_df <- glb_entity_df[sample(1:nrow(glb_entity_df), nrow(glb_entity_df) / 1000),]
    glb_predct_df <- subset(glb_entity_df, Year > 2006)
    comment(glb_predct_df) <- "glb_predct_df"
    myprint_df(glb_predct_df)
    str(glb_predct_df)
}         
```

```
##     Year Month    MEI    CO2     CH4     N2O  CFC.11  CFC.12      TSI
## 285 2007     1  0.974 382.93 1799.66 320.561 248.372 539.206 1365.717
## 286 2007     2  0.510 383.81 1803.08 320.571 248.264 538.973 1365.715
## 287 2007     3  0.074 384.56 1803.10 320.548 247.997 538.811 1365.754
## 288 2007     4 -0.049 386.40 1802.11 320.518 247.574 538.586 1365.723
## 289 2007     5  0.183 386.58 1795.65 320.445 247.224 538.130 1365.693
## 290 2007     6 -0.358 386.05 1781.81 320.332 246.881 537.376 1365.762
##     Aerosols  Temp
## 285   0.0054 0.601
## 286   0.0051 0.498
## 287   0.0045 0.435
## 288   0.0045 0.466
## 289   0.0041 0.372
## 290   0.0040 0.382
##     Year Month    MEI    CO2     CH4     N2O  CFC.11  CFC.12      TSI
## 285 2007     1  0.974 382.93 1799.66 320.561 248.372 539.206 1365.717
## 287 2007     3  0.074 384.56 1803.10 320.548 247.997 538.811 1365.754
## 292 2007     8 -0.440 382.00 1779.38 320.471 246.307 537.125 1365.757
## 296 2007    12 -1.168 383.89 1805.58 321.217 246.261 537.052 1365.693
## 301 2008     5 -0.355 388.50 1796.43 321.420 244.914 535.399 1365.717
## 305 2008     9 -0.643 383.09 1795.08 321.529 244.083 535.048 1365.665
##     Aerosols  Temp
## 285   0.0054 0.601
## 287   0.0045 0.435
## 292   0.0041 0.358
## 296   0.0040 0.226
## 301   0.0031 0.283
## 305   0.0043 0.378
##     Year Month    MEI    CO2     CH4     N2O  CFC.11  CFC.12      TSI
## 303 2008     7  0.003 386.42 1782.93 321.372 244.434 535.026 1365.672
## 304 2008     8 -0.266 384.15 1779.88 321.405 244.200 535.072 1365.657
## 305 2008     9 -0.643 383.09 1795.08 321.529 244.083 535.048 1365.665
## 306 2008    10 -0.780 382.99 1814.18 321.796 244.080 534.927 1365.676
## 307 2008    11 -0.621 384.13 1812.37 322.013 244.225 534.906 1365.707
## 308 2008    12 -0.666 385.56 1812.88 322.182 244.204 535.005 1365.693
##     Aerosols  Temp
## 303   0.0033 0.406
## 304   0.0036 0.407
## 305   0.0043 0.378
## 306   0.0046 0.440
## 307   0.0048 0.394
## 308   0.0046 0.330
## 'data.frame':	24 obs. of  11 variables:
##  $ Year    : int  2007 2007 2007 2007 2007 2007 2007 2007 2007 2007 ...
##  $ Month   : int  1 2 3 4 5 6 7 8 9 10 ...
##  $ MEI     : num  0.974 0.51 0.074 -0.049 0.183 ...
##  $ CO2     : num  383 384 385 386 387 ...
##  $ CH4     : num  1800 1803 1803 1802 1796 ...
##  $ N2O     : num  321 321 321 321 320 ...
##  $ CFC.11  : num  248 248 248 248 247 ...
##  $ CFC.12  : num  539 539 539 539 538 ...
##  $ TSI     : num  1366 1366 1366 1366 1366 ...
##  $ Aerosols: num  0.0054 0.0051 0.0045 0.0045 0.0041 0.004 0.004 0.0041 0.0042 0.0041 ...
##  $ Temp    : num  0.601 0.498 0.435 0.466 0.372 0.382 0.394 0.358 0.402 0.362 ...
##  - attr(*, "comment")= chr "glb_predct_df"
```

```r
glb_entity_df <- subset(glb_entity_df, Year <= 2006)
print(dim(glb_entity_df))
```

```
## [1] 284  11
```

```r
script_df <- rbind(script_df,
                   data.frame(chunk_label="cleanse_data", 
                              chunk_step_major=max(script_df$chunk_step_major)+1, 
                              chunk_step_minor=0))
print(script_df)
```

```
##    chunk_label chunk_step_major chunk_step_minor
## 1  import_data                1                0
## 2 cleanse_data                2                0
```

## Step `2`: cleanse data

```r
script_df <- rbind(script_df, 
                   data.frame(chunk_label="inspect_explore_data", 
                              chunk_step_major=max(script_df$chunk_step_major), 
                              chunk_step_minor=1))
print(script_df)
```

```
##            chunk_label chunk_step_major chunk_step_minor
## 1          import_data                1                0
## 2         cleanse_data                2                0
## 3 inspect_explore_data                2                1
```

### Step `2`.`1`: inspect/explore data

```r
#print(str(glb_entity_df))
#View(glb_entity_df)

# List info gathered for various columns
# <col_name>:   <description>; <notes>

# Year:         the observation year.
# Month:        the observation month.
# Temp:         the difference in degrees Celsius between the average global 
#                     temperature in that period and a reference value. 
#                     Source: Climatic Research Unit at the University of East Anglia.

# CO2, N2O, CH4, CFC.11, CFC.12: atmospheric concentrations of 
#                 carbon dioxide (CO2), 
#                 nitrous oxide (N2O), 
#                 methane  (CH4), 
#                 trichlorofluoromethane (CCl3F; commonly referred to as CFC-11) 
#                 dichlorodifluoromethane (CCl2F2; commonly referred to as CFC-12)
#                 Source: ESRL/NOAA Global Monitoring Division.

# CO2, N2O and CH4 are expressed in ppmv (parts per million by volume  
#     -- i.e., 397 ppmv of CO2 means that CO2 constitutes 397 millionths of the 
#     total volume of the atmosphere)
# CFC.11 and CFC.12 are expressed in ppbv (parts per billion by volume). 

# Aerosols:     the mean stratospheric aerosol optical depth at 550 nm. 
#                 This variable is linked to volcanoes, as volcanic eruptions 
#                 result in new particles being added to the atmosphere, which 
#                 affect how much of the sun's energy is reflected back into space.
#                 Source: Godard Institute for Space Studies at NASA.

# TSI:  the total solar irradiance (TSI) in W/m2 (the rate at which the sun's 
#         energy is deposited per unit area). Due to sunspots and other solar 
#         phenomena, the amount of energy that is given off by the sun varies substantially
#         with time. Source: SOLARIS-HEPPA project website.

# MEI:  multivariate El Nino Southern Oscillation index (MEI), 
#         a measure of the strength of the El Nino/La Nina-Southern Oscillation 
#         (a weather effect in the Pacific Ocean that affects global temperatures)
#         Source: ESRL/NOAA Physical Sciences Division.

# Create new features that help diagnostics
#   Convert factors to dummy variables
#   Build splines   require(splines); bsBasis <- bs(training$age, df=3)

add_new_diag_feats <- function(obs_df, obs_twin_df) {
    obs_df <- mutate(obs_df,
#         <col_name>.NA=is.na(<col_name>)

#         <col_name>.fctr=factor(<col_name>, 
#                     as.factor(union(obs_df$<col_name>, obs_twin_df$<col_name>))) 

          # This doesn't work - use sapply instead
#         <col_name>.fctr_num=grep(<col_name>, levels(<col_name>.fctr)), 
#         
#         Date.my=as.Date(strptime(Date, "%m/%d/%y %H:%M")),
#         Year=year(Date.my),
#         Month=months(Date.my),
#         Weekday=weekdays(Date.my)
        
                        )

    # If levels of a factor are different across obs_df & glb_predct_df; predict.glm fails  
    # Transformations not handled by mutate
#     obs_df$<col_name>.fctr.num <- sapply(1:nrow(obs_df), 
#         function(row_ix) grep(obs_df[row_ix, "<col_name>"],
#                               levels(obs_df[row_ix, "<col_name>.fctr"])))
    
    print(summary(obs_df))
    print(sapply(names(obs_df), function(col) sum(is.na(obs_df[, col]))))
    return(obs_df)
}

# glb_entity_df <- add_new_diag_feats(glb_entity_df, glb_predct_df)
# glb_predct_df <- add_new_diag_feats(glb_predct_df, glb_entity_df)

#pairs(subset(glb_entity_df, select=-c(col_symbol)))

#   Histogram of predictor in glb_entity_df & glb_predct_df
# Check for glb_predct_df & glb_entity_df features range mismatches

# Other diagnostics:
# print(subset(glb_entity_df, <col1_name> == max(glb_entity_df$<col1_name>, na.rm=TRUE) & 
#                         <col2_name> <= mean(glb_entity_df$<col1_name>, na.rm=TRUE)))

# print(<col_name>_freq_glb_entity_df <- mycreate_tbl_df(glb_entity_df, "<col_name>"))
# print(which.min(table(glb_entity_df$<col_name>)))
# print(which.max(table(glb_entity_df$<col_name>)))
# print(which.max(table(glb_entity_df$<col1_name>, glb_entity_df$<col2_name>)[, 2]))
# print(table(glb_entity_df$<col1_name>, glb_entity_df$<col2_name>))
# print(table(is.na(glb_entity_df$<col1_name>), glb_entity_df$<col2_name>))
# print(xtabs(~ <col1_name>, glb_entity_df))
# print(xtabs(~ <col1_name> + <col2_name>, glb_entity_df))
# print(<col1_name>_<col2_name>_xtab_glb_entity_df <- 
#   mycreate_xtab(glb_entity_df, c("<col1_name>", "<col2_name>")))
# <col1_name>_<col2_name>_xtab_glb_entity_df[is.na(<col1_name>_<col2_name>_xtab_glb_entity_df)] <- 0
# print(<col1_name>_<col2_name>_xtab_glb_entity_df <- 
#   mutate(<col1_name>_<col2_name>_xtab_glb_entity_df, 
#             <col3_name>=(<col1_name> * 1.0) / (<col1_name> + <col2_name>))) 

# print(<col2_name>_min_entity_arr <- 
#    sort(tapply(glb_entity_df$<col1_name>, glb_entity_df$<col2_name>, min, na.rm=TRUE)))
# print(<col1_name>_na_by_<col2_name>_arr <- 
#    sort(tapply(glb_entity_df$<col1_name>.NA, glb_entity_df$<col2_name>, mean, na.rm=TRUE)))


# Other plots:
# print(myplot_histogram(glb_entity_df, "<col1_name>"))
# print(myplot_box(df=glb_entity_df, ycol_names="<col1_name>"))
# print(myplot_box(df=glb_entity_df, ycol_names="<col1_name>", xcol_name="<col2_name>"))
# print(myplot_line(subset(glb_entity_df, Symbol %in% c("KO", "PG")), 
#                   "Date.my", "StockPrice", facet_row_colnames="Symbol") + 
#     geom_vline(xintercept=as.numeric(as.Date("2003-03-01"))) +
#     geom_vline(xintercept=as.numeric(as.Date("1983-01-01")))        
#         )
# print(myplot_scatter(glb_entity_df, "<col1_name>", "<col2_name>"))

script_df <- rbind(script_df, 
    data.frame(chunk_label="manage_missing_data", 
        chunk_step_major=max(script_df$chunk_step_major), 
        chunk_step_minor=script_df[nrow(script_df), "chunk_step_minor"]+1))
print(script_df)
```

```
##            chunk_label chunk_step_major chunk_step_minor
## 1          import_data                1                0
## 2         cleanse_data                2                0
## 3 inspect_explore_data                2                1
## 4  manage_missing_data                2                2
```

### Step `2`.`2`: manage missing data

```r
script_df <- rbind(script_df, 
    data.frame(chunk_label="encode_retype_data", 
        chunk_step_major=max(script_df$chunk_step_major), 
        chunk_step_minor=script_df[nrow(script_df), "chunk_step_minor"]+1))
print(script_df)
```

```
##            chunk_label chunk_step_major chunk_step_minor
## 1          import_data                1                0
## 2         cleanse_data                2                0
## 3 inspect_explore_data                2                1
## 4  manage_missing_data                2                2
## 5   encode_retype_data                2                3
```

### Step `2`.`3`: encode/retype data

```r
# map_<col_name>_df <- myimport_data(
#     url="<map_url>", 
#     comment="map_<col_name>_df", print_diagn=TRUE)
# map_<col_name>_df <- read.csv(paste0(getwd(), "/data/<file_name>.csv"), strip.white=TRUE)

# glb_entity_df <- mymap_codes(glb_entity_df, "<from_col_name>", "<to_col_name>", 
#     map_<to_col_name>_df, map_join_col_name="<map_join_col_name>", 
#                           map_tgt_col_name="<to_col_name>")
# glb_predct_df <- mymap_codes(glb_predct_df, "<from_col_name>", "<to_col_name>", 
#     map_<to_col_name>_df, map_join_col_name="<map_join_col_name>", 
#                           map_tgt_col_name="<to_col_name>")
    					
# glb_entity_df$<col_name>.fctr <- factor(glb_entity_df$<col_name>, 
#                     as.factor(union(glb_entity_df$<col_name>, glb_predct_df$<col_name>)))
# glb_predct_df$<col_name>.fctr <- factor(glb_predct_df$<col_name>, 
#                     as.factor(union(glb_entity_df$<col_name>, glb_predct_df$<col_name>)))

script_df <- rbind(script_df, 
                   data.frame(chunk_label="extract_features", 
                              chunk_step_major=max(script_df$chunk_step_major)+1, 
                              chunk_step_minor=0))
print(script_df)
```

```
##            chunk_label chunk_step_major chunk_step_minor
## 1          import_data                1                0
## 2         cleanse_data                2                0
## 3 inspect_explore_data                2                1
## 4  manage_missing_data                2                2
## 5   encode_retype_data                2                3
## 6     extract_features                3                0
```

## Step `3`: extract features

```r
# Create new features that help prediction
# glb_entity_df <- mutate(glb_entity_df,
#     <new_col_name>=
#                     )

# glb_predct_df <- mutate(glb_predct_df,
#     <new_col_name>=
#                     )

# print(summary(glb_entity_df))
# print(summary(glb_predct_df))

script_df <- rbind(script_df, 
                   data.frame(chunk_label="select_features", 
                              chunk_step_major=max(script_df$chunk_step_major)+1, 
                              chunk_step_minor=0))
print(script_df)
```

```
##            chunk_label chunk_step_major chunk_step_minor
## 1          import_data                1                0
## 2         cleanse_data                2                0
## 3 inspect_explore_data                2                1
## 4  manage_missing_data                2                2
## 5   encode_retype_data                2                3
## 6     extract_features                3                0
## 7      select_features                4                0
```

## Step `4`: select features

```r
print(glb_feats_df <- myselect_features())
```

```
##                id      cor.y cor.y.abs
## CO2           CO2  0.7885292 0.7885292
## N2O           N2O  0.7786389 0.7786389
## CH4           CH4  0.7032550 0.7032550
## CFC.12     CFC.12  0.6875575 0.6875575
## CFC.11     CFC.11  0.4077103 0.4077103
## Aerosols Aerosols -0.3849137 0.3849137
## TSI           TSI  0.2433827 0.2433827
## MEI           MEI  0.1724708 0.1724708
```

```r
script_df <- rbind(script_df, 
    data.frame(chunk_label="remove_correlated_features", 
        chunk_step_major=max(script_df$chunk_step_major),
        chunk_step_minor=script_df[nrow(script_df), "chunk_step_minor"]+1))        
print(script_df)
```

```
##                  chunk_label chunk_step_major chunk_step_minor
## 1                import_data                1                0
## 2               cleanse_data                2                0
## 3       inspect_explore_data                2                1
## 4        manage_missing_data                2                2
## 5         encode_retype_data                2                3
## 6           extract_features                3                0
## 7            select_features                4                0
## 8 remove_correlated_features                4                1
```

### Step `4`.`1`: remove correlated features

```r
print(glb_feats_df <- orderBy(~-cor.y, 
                    merge(glb_feats_df, mydelete_cor_features(), all.x=TRUE)))
```

```
##                  CO2         N2O        CH4       CFC.12      CFC.11
## CO2       1.00000000  0.97671982  0.8772796  0.852689627  0.51405975
## N2O       0.97671982  1.00000000  0.8998386  0.867930776  0.52247732
## CH4       0.87727963  0.89983864  1.0000000  0.963616248  0.77990402
## CFC.12    0.85268963  0.86793078  0.9636162  1.000000000  0.86898518
## CFC.11    0.51405975  0.52247732  0.7799040  0.868985183  1.00000000
## Aerosols -0.35615480 -0.33705457 -0.2678092 -0.225131244 -0.04392120
## TSI       0.17742893  0.19975668  0.2455284  0.255302814  0.27204596
## MEI      -0.04114717 -0.05081978 -0.0334193  0.008285544  0.06900044
##             Aerosols         TSI          MEI
## CO2      -0.35615480  0.17742893 -0.041147165
## N2O      -0.33705457  0.19975668 -0.050819775
## CH4      -0.26780919  0.24552844 -0.033419301
## CFC.12   -0.22513124  0.25530281  0.008285544
## CFC.11   -0.04392120  0.27204596  0.069000439
## Aerosols  1.00000000  0.05211651  0.340237787
## TSI       0.05211651  1.00000000 -0.154491923
## MEI       0.34023779 -0.15449192  1.000000000
##                 CO2        N2O       CH4      CFC.12     CFC.11   Aerosols
## CO2      0.00000000 0.97671982 0.8772796 0.852689627 0.51405975 0.35615480
## N2O      0.97671982 0.00000000 0.8998386 0.867930776 0.52247732 0.33705457
## CH4      0.87727963 0.89983864 0.0000000 0.963616248 0.77990402 0.26780919
## CFC.12   0.85268963 0.86793078 0.9636162 0.000000000 0.86898518 0.22513124
## CFC.11   0.51405975 0.52247732 0.7799040 0.868985183 0.00000000 0.04392120
## Aerosols 0.35615480 0.33705457 0.2678092 0.225131244 0.04392120 0.00000000
## TSI      0.17742893 0.19975668 0.2455284 0.255302814 0.27204596 0.05211651
## MEI      0.04114717 0.05081978 0.0334193 0.008285544 0.06900044 0.34023779
##                 TSI         MEI
## CO2      0.17742893 0.041147165
## N2O      0.19975668 0.050819775
## CH4      0.24552844 0.033419301
## CFC.12   0.25530281 0.008285544
## CFC.11   0.27204596 0.069000439
## Aerosols 0.05211651 0.340237787
## TSI      0.00000000 0.154491923
## MEI      0.15449192 0.000000000
## [1] "cor(CO2, N2O)=0.9767"
```

![](glb_Climate_files/figure-html/remove_correlated_features-1.png) 

```
## [1] "cor(Temp, CO2)=0.7885"
## [1] "cor(Temp, N2O)=0.7786"
```

```
## geom_smooth: method="auto" and size of largest group is <1000, so using loess. Use 'method = x' to change the smoothing method.
## geom_smooth: method="auto" and size of largest group is <1000, so using loess. Use 'method = x' to change the smoothing method.
```

```
## Warning in mydelete_cor_features(): Dropping N2O as a feature
```

![](glb_Climate_files/figure-html/remove_correlated_features-2.png) 

```
##                id      cor.y cor.y.abs
## CO2           CO2  0.7885292 0.7885292
## CH4           CH4  0.7032550 0.7032550
## CFC.12     CFC.12  0.6875575 0.6875575
## CFC.11     CFC.11  0.4077103 0.4077103
## Aerosols Aerosols -0.3849137 0.3849137
## TSI           TSI  0.2433827 0.2433827
## MEI           MEI  0.1724708 0.1724708
##                  CO2        CH4       CFC.12      CFC.11    Aerosols
## CO2       1.00000000  0.8772796  0.852689627  0.51405975 -0.35615480
## CH4       0.87727963  1.0000000  0.963616248  0.77990402 -0.26780919
## CFC.12    0.85268963  0.9636162  1.000000000  0.86898518 -0.22513124
## CFC.11    0.51405975  0.7799040  0.868985183  1.00000000 -0.04392120
## Aerosols -0.35615480 -0.2678092 -0.225131244 -0.04392120  1.00000000
## TSI       0.17742893  0.2455284  0.255302814  0.27204596  0.05211651
## MEI      -0.04114717 -0.0334193  0.008285544  0.06900044  0.34023779
##                  TSI          MEI
## CO2       0.17742893 -0.041147165
## CH4       0.24552844 -0.033419301
## CFC.12    0.25530281  0.008285544
## CFC.11    0.27204596  0.069000439
## Aerosols  0.05211651  0.340237787
## TSI       1.00000000 -0.154491923
## MEI      -0.15449192  1.000000000
##                 CO2       CH4      CFC.12     CFC.11   Aerosols        TSI
## CO2      0.00000000 0.8772796 0.852689627 0.51405975 0.35615480 0.17742893
## CH4      0.87727963 0.0000000 0.963616248 0.77990402 0.26780919 0.24552844
## CFC.12   0.85268963 0.9636162 0.000000000 0.86898518 0.22513124 0.25530281
## CFC.11   0.51405975 0.7799040 0.868985183 0.00000000 0.04392120 0.27204596
## Aerosols 0.35615480 0.2678092 0.225131244 0.04392120 0.00000000 0.05211651
## TSI      0.17742893 0.2455284 0.255302814 0.27204596 0.05211651 0.00000000
## MEI      0.04114717 0.0334193 0.008285544 0.06900044 0.34023779 0.15449192
##                  MEI
## CO2      0.041147165
## CH4      0.033419301
## CFC.12   0.008285544
## CFC.11   0.069000439
## Aerosols 0.340237787
## TSI      0.154491923
## MEI      0.000000000
## [1] "cor(CH4, CFC.12)=0.9636"
```

![](glb_Climate_files/figure-html/remove_correlated_features-3.png) 

```
## [1] "cor(Temp, CH4)=0.7033"
## [1] "cor(Temp, CFC.12)=0.6876"
```

```
## geom_smooth: method="auto" and size of largest group is <1000, so using loess. Use 'method = x' to change the smoothing method.
## geom_smooth: method="auto" and size of largest group is <1000, so using loess. Use 'method = x' to change the smoothing method.
```

```
## Warning in mydelete_cor_features(): Dropping CFC.12 as a feature
```

![](glb_Climate_files/figure-html/remove_correlated_features-4.png) 

```
##                id      cor.y cor.y.abs
## CO2           CO2  0.7885292 0.7885292
## CH4           CH4  0.7032550 0.7032550
## CFC.11     CFC.11  0.4077103 0.4077103
## Aerosols Aerosols -0.3849137 0.3849137
## TSI           TSI  0.2433827 0.2433827
## MEI           MEI  0.1724708 0.1724708
##                  CO2        CH4      CFC.11    Aerosols         TSI
## CO2       1.00000000  0.8772796  0.51405975 -0.35615480  0.17742893
## CH4       0.87727963  1.0000000  0.77990402 -0.26780919  0.24552844
## CFC.11    0.51405975  0.7799040  1.00000000 -0.04392120  0.27204596
## Aerosols -0.35615480 -0.2678092 -0.04392120  1.00000000  0.05211651
## TSI       0.17742893  0.2455284  0.27204596  0.05211651  1.00000000
## MEI      -0.04114717 -0.0334193  0.06900044  0.34023779 -0.15449192
##                  MEI
## CO2      -0.04114717
## CH4      -0.03341930
## CFC.11    0.06900044
## Aerosols  0.34023779
## TSI      -0.15449192
## MEI       1.00000000
##                 CO2       CH4     CFC.11   Aerosols        TSI        MEI
## CO2      0.00000000 0.8772796 0.51405975 0.35615480 0.17742893 0.04114717
## CH4      0.87727963 0.0000000 0.77990402 0.26780919 0.24552844 0.03341930
## CFC.11   0.51405975 0.7799040 0.00000000 0.04392120 0.27204596 0.06900044
## Aerosols 0.35615480 0.2678092 0.04392120 0.00000000 0.05211651 0.34023779
## TSI      0.17742893 0.2455284 0.27204596 0.05211651 0.00000000 0.15449192
## MEI      0.04114717 0.0334193 0.06900044 0.34023779 0.15449192 0.00000000
## [1] "cor(CO2, CH4)=0.8773"
```

![](glb_Climate_files/figure-html/remove_correlated_features-5.png) 

```
## [1] "cor(Temp, CO2)=0.7885"
## [1] "cor(Temp, CH4)=0.7033"
```

```
## geom_smooth: method="auto" and size of largest group is <1000, so using loess. Use 'method = x' to change the smoothing method.
## geom_smooth: method="auto" and size of largest group is <1000, so using loess. Use 'method = x' to change the smoothing method.
```

```
## Warning in mydelete_cor_features(): Dropping CH4 as a feature
```

![](glb_Climate_files/figure-html/remove_correlated_features-6.png) 

```
##                id      cor.y cor.y.abs
## CO2           CO2  0.7885292 0.7885292
## CFC.11     CFC.11  0.4077103 0.4077103
## Aerosols Aerosols -0.3849137 0.3849137
## TSI           TSI  0.2433827 0.2433827
## MEI           MEI  0.1724708 0.1724708
##                  CO2      CFC.11    Aerosols         TSI         MEI
## CO2       1.00000000  0.51405975 -0.35615480  0.17742893 -0.04114717
## CFC.11    0.51405975  1.00000000 -0.04392120  0.27204596  0.06900044
## Aerosols -0.35615480 -0.04392120  1.00000000  0.05211651  0.34023779
## TSI       0.17742893  0.27204596  0.05211651  1.00000000 -0.15449192
## MEI      -0.04114717  0.06900044  0.34023779 -0.15449192  1.00000000
##                 CO2     CFC.11   Aerosols        TSI        MEI
## CO2      0.00000000 0.51405975 0.35615480 0.17742893 0.04114717
## CFC.11   0.51405975 0.00000000 0.04392120 0.27204596 0.06900044
## Aerosols 0.35615480 0.04392120 0.00000000 0.05211651 0.34023779
## TSI      0.17742893 0.27204596 0.05211651 0.00000000 0.15449192
## MEI      0.04114717 0.06900044 0.34023779 0.15449192 0.00000000
##         id      cor.y cor.y.abs cor.low
## 5      CO2  0.7885292 0.7885292       1
## 7      N2O  0.7786389 0.7786389      NA
## 4      CH4  0.7032550 0.7032550      NA
## 3   CFC.12  0.6875575 0.6875575      NA
## 2   CFC.11  0.4077103 0.4077103       1
## 8      TSI  0.2433827 0.2433827       1
## 6      MEI  0.1724708 0.1724708       1
## 1 Aerosols -0.3849137 0.3849137       1
```

```r
script_df <- rbind(script_df, 
                   data.frame(chunk_label="run_models", 
                              chunk_step_major=max(script_df$chunk_step_major)+1, 
                              chunk_step_minor=0))
print(script_df)
```

```
##                  chunk_label chunk_step_major chunk_step_minor
## 1                import_data                1                0
## 2               cleanse_data                2                0
## 3       inspect_explore_data                2                1
## 4        manage_missing_data                2                2
## 5         encode_retype_data                2                3
## 6           extract_features                3                0
## 7            select_features                4                0
## 8 remove_correlated_features                4                1
## 9                 run_models                5                0
```

## Step `5`: run models

```r
max_cor_y_x_var <- subset(glb_feats_df, cor.low == 1)[1, "id"]

#   Regression:
if (glb_is_regression) {
    #   Linear:
    myrun_mdl_fn <- myrun_mdl_lm
}    

#   Classification:
if (glb_is_classification) {
    #   Logit Regression:
    myrun_mdl_fn <- myrun_mdl_glm
}    
    
# Highest cor.y
ret_lst <- myrun_mdl_fn(indep_vars_vctr=max_cor_y_x_var,
                        fit_df=glb_entity_df, OOB_df=glb_predct_df)
```

```
## [1] 1.050143
## [1] -0.7919957
## 
## Call:
## lm(formula = reformulate(indep_vars_vctr, response = glb_predct_var), 
##     data = fit_df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.31912 -0.07819 -0.00129  0.05805  0.43420 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -4.2646515  0.2096815  -20.34   <2e-16 ***
## CO2          0.0124855  0.0005799   21.53   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.1116 on 282 degrees of freedom
## Multiple R-squared:  0.6218,	Adjusted R-squared:  0.6204 
## F-statistic: 463.6 on 1 and 282 DF,  p-value: < 2.2e-16
## 
##   feats n.fit  R.sq.fit   R.sq.OOB Adj.R.sq.fit  SSE.fit  SSE.OOB
## 1   CO2   284 0.6217783 -0.7919957    0.6217783 3.511885 1.050143
##   f.score.OOB
## 1          NA
```

```r
# Enhance Highest cor.y model with additions of interaction terms that were 
#   dropped due to high correlations
ret_lst <- myrun_mdl_fn(indep_vars_vctr=c(max_cor_y_x_var, 
    paste(max_cor_y_x_var, subset(glb_feats_df, is.na(cor.low))[, "id"], sep=":")),
                        fit_df=glb_entity_df, OOB_df=glb_predct_df)    
```

```
## [1] 0.9795889
## [1] -0.6715997
## 
## Call:
## lm(formula = reformulate(indep_vars_vctr, response = glb_predct_var), 
##     data = fit_df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.31421 -0.07546 -0.00240  0.05969  0.43026 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)   
## (Intercept) -3.208e+00  9.944e-01  -3.226  0.00141 **
## CO2          4.950e-03  8.255e-03   0.600  0.54922   
## CO2:N2O      1.660e-05  1.918e-05   0.866  0.38740   
## CO2:CH4     -5.300e-07  1.718e-06  -0.309  0.75793   
## CO2:CFC.12   7.268e-07  1.188e-06   0.612  0.54105   
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.1119 on 279 degrees of freedom
## Multiple R-squared:  0.6237,	Adjusted R-squared:  0.6183 
## F-statistic: 115.6 on 4 and 279 DF,  p-value: < 2.2e-16
## 
##                               feats n.fit  R.sq.fit   R.sq.OOB
## 2 CO2, CO2:N2O, CO2:CH4, CO2:CFC.12   284 0.6237367 -0.6715997
## 1                               CO2   284 0.6217783 -0.7919957
##   Adj.R.sq.fit  SSE.fit   SSE.OOB f.score.OOB
## 2    0.6237367 3.493701 0.9795889          NA
## 1    0.6217783 3.511885 1.0501433          NA
```

```r
# Low correlated X
ret_lst <- myrun_mdl_fn(indep_vars_vctr=subset(glb_feats_df, cor.low == 1)[, "id"],
                        fit_df=glb_entity_df, OOB_df=glb_predct_df)
```

```
## [1] 0.3752535
## [1] 0.3596563
## 
## Call:
## lm(formula = reformulate(indep_vars_vctr, response = glb_predct_var), 
##     data = fit_df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.26718 -0.06130 -0.00850  0.06045  0.34863 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -1.226e+02  2.038e+01  -6.016 5.62e-09 ***
## CO2          1.097e-02  6.228e-04  17.621  < 2e-16 ***
## CFC.11      -2.987e-04  3.235e-04  -0.923    0.357    
## TSI          8.710e-02  1.495e-02   5.827 1.56e-08 ***
## MEI          6.260e-02  6.603e-03   9.481  < 2e-16 ***
## Aerosols    -1.563e+00  2.183e-01  -7.161 7.19e-12 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.09418 on 278 degrees of freedom
## Multiple R-squared:  0.7345,	Adjusted R-squared:  0.7297 
## F-statistic: 153.8 on 5 and 278 DF,  p-value: < 2.2e-16
## 
##                               feats n.fit  R.sq.fit   R.sq.OOB
## 3   CO2, CFC.11, TSI, MEI, Aerosols   284 0.7344547  0.3596563
## 2 CO2, CO2:N2O, CO2:CH4, CO2:CFC.12   284 0.6237367 -0.6715997
## 1                               CO2   284 0.6217783 -0.7919957
##   Adj.R.sq.fit  SSE.fit   SSE.OOB f.score.OOB
## 3    0.7344547 2.465656 0.3752535          NA
## 2    0.6237367 3.493701 0.9795889          NA
## 1    0.6217783 3.511885 1.0501433          NA
```

```r
glb_sel_mdl <- glb_mdl                        

# All X that is not user excluded
ret_lst <- myrun_mdl_fn(indep_vars_vctr=setdiff(setdiff(names(glb_entity_df),
                                                        glb_predct_var),
                                                glb_exclude_vars_as_features),
                        fit_df=glb_entity_df, OOB_df=glb_predct_df)
```

```
## [1] 0.2183475
## [1] 0.6274054
## 
## Call:
## lm(formula = reformulate(indep_vars_vctr, response = glb_predct_var), 
##     data = fit_df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.25888 -0.05913 -0.00082  0.05649  0.32433 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -1.246e+02  1.989e+01  -6.265 1.43e-09 ***
## MEI          6.421e-02  6.470e-03   9.923  < 2e-16 ***
## CO2          6.457e-03  2.285e-03   2.826  0.00505 ** 
## CH4          1.240e-04  5.158e-04   0.240  0.81015    
## N2O         -1.653e-02  8.565e-03  -1.930  0.05467 .  
## CFC.11      -6.631e-03  1.626e-03  -4.078 5.96e-05 ***
## CFC.12       3.808e-03  1.014e-03   3.757  0.00021 ***
## TSI          9.314e-02  1.475e-02   6.313 1.10e-09 ***
## Aerosols    -1.538e+00  2.133e-01  -7.210 5.41e-12 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.09171 on 275 degrees of freedom
## Multiple R-squared:  0.7509,	Adjusted R-squared:  0.7436 
## F-statistic: 103.6 on 8 and 275 DF,  p-value: < 2.2e-16
## 
##                                               feats n.fit  R.sq.fit
## 4 MEI, CO2, CH4, N2O, CFC.11, CFC.12, TSI, Aerosols   284 0.7508933
## 3                   CO2, CFC.11, TSI, MEI, Aerosols   284 0.7344547
## 2                 CO2, CO2:N2O, CO2:CH4, CO2:CFC.12   284 0.6237367
## 1                                               CO2   284 0.6217783
##     R.sq.OOB Adj.R.sq.fit  SSE.fit   SSE.OOB f.score.OOB
## 4  0.6274054    0.7508933 2.313020 0.2183475          NA
## 3  0.3596563    0.7344547 2.465656 0.3752535          NA
## 2 -0.6715997    0.6237367 3.493701 0.9795889          NA
## 1 -0.7919957    0.6217783 3.511885 1.0501433          NA
```

```r
# User specified
# Problem 1.1: MEI, CO2, CH4, N2O, CFC.11, CFC.12, TSI, and Aerosols
ret_lst <- myrun_mdl_fn(indep_vars_vctr=c("MEI", "CO2", "CH4", "N2O", 
                                          "CFC.11", "CFC.12", "TSI", "Aerosols"),
                        fit_df=glb_entity_df, OOB_df=glb_predct_df)
```

```
## [1] 0.2183475
## [1] 0.6274054
## 
## Call:
## lm(formula = reformulate(indep_vars_vctr, response = glb_predct_var), 
##     data = fit_df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.25888 -0.05913 -0.00082  0.05649  0.32433 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -1.246e+02  1.989e+01  -6.265 1.43e-09 ***
## MEI          6.421e-02  6.470e-03   9.923  < 2e-16 ***
## CO2          6.457e-03  2.285e-03   2.826  0.00505 ** 
## CH4          1.240e-04  5.158e-04   0.240  0.81015    
## N2O         -1.653e-02  8.565e-03  -1.930  0.05467 .  
## CFC.11      -6.631e-03  1.626e-03  -4.078 5.96e-05 ***
## CFC.12       3.808e-03  1.014e-03   3.757  0.00021 ***
## TSI          9.314e-02  1.475e-02   6.313 1.10e-09 ***
## Aerosols    -1.538e+00  2.133e-01  -7.210 5.41e-12 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.09171 on 275 degrees of freedom
## Multiple R-squared:  0.7509,	Adjusted R-squared:  0.7436 
## F-statistic: 103.6 on 8 and 275 DF,  p-value: < 2.2e-16
## 
##                                               feats n.fit  R.sq.fit
## 4 MEI, CO2, CH4, N2O, CFC.11, CFC.12, TSI, Aerosols   284 0.7508933
## 5 MEI, CO2, CH4, N2O, CFC.11, CFC.12, TSI, Aerosols   284 0.7508933
## 3                   CO2, CFC.11, TSI, MEI, Aerosols   284 0.7344547
## 2                 CO2, CO2:N2O, CO2:CH4, CO2:CFC.12   284 0.6237367
## 1                                               CO2   284 0.6217783
##     R.sq.OOB Adj.R.sq.fit  SSE.fit   SSE.OOB f.score.OOB
## 4  0.6274054    0.7508933 2.313020 0.2183475          NA
## 5  0.6274054    0.7508933 2.313020 0.2183475          NA
## 3  0.3596563    0.7344547 2.465656 0.3752535          NA
## 2 -0.6715997    0.6237367 3.493701 0.9795889          NA
## 1 -0.7919957    0.6217783 3.511885 1.0501433          NA
```

```r
P1.1_mdl <- glb_mdl
# Problem 3.0: MEI, N2O, TSI, and Aerosols
ret_lst <- myrun_mdl_fn(indep_vars_vctr=c("MEI", "N2O", "TSI", "Aerosols"),
                        fit_df=glb_entity_df, OOB_df=glb_predct_df)
```

```
## [1] 0.2948967
## [1] 0.4967795
## 
## Call:
## lm(formula = reformulate(indep_vars_vctr, response = glb_predct_var), 
##     data = fit_df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.27916 -0.05975 -0.00595  0.05672  0.34195 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -1.162e+02  2.022e+01  -5.747 2.37e-08 ***
## MEI          6.419e-02  6.652e-03   9.649  < 2e-16 ***
## N2O          2.532e-02  1.311e-03  19.307  < 2e-16 ***
## TSI          7.949e-02  1.487e-02   5.344 1.89e-07 ***
## Aerosols    -1.702e+00  2.180e-01  -7.806 1.19e-13 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.09547 on 279 degrees of freedom
## Multiple R-squared:  0.7261,	Adjusted R-squared:  0.7222 
## F-statistic: 184.9 on 4 and 279 DF,  p-value: < 2.2e-16
## 
##                                               feats n.fit  R.sq.fit
## 4 MEI, CO2, CH4, N2O, CFC.11, CFC.12, TSI, Aerosols   284 0.7508933
## 5 MEI, CO2, CH4, N2O, CFC.11, CFC.12, TSI, Aerosols   284 0.7508933
## 6                           MEI, N2O, TSI, Aerosols   284 0.7261321
## 3                   CO2, CFC.11, TSI, MEI, Aerosols   284 0.7344547
## 2                 CO2, CO2:N2O, CO2:CH4, CO2:CFC.12   284 0.6237367
## 1                                               CO2   284 0.6217783
##     R.sq.OOB Adj.R.sq.fit  SSE.fit   SSE.OOB f.score.OOB
## 4  0.6274054    0.7508933 2.313020 0.2183475          NA
## 5  0.6274054    0.7508933 2.313020 0.2183475          NA
## 6  0.4967795    0.7261321 2.542933 0.2948967          NA
## 3  0.3596563    0.7344547 2.465656 0.3752535          NA
## 2 -0.6715997    0.6237367 3.493701 0.9795889          NA
## 1 -0.7919957    0.6217783 3.511885 1.0501433          NA
```

```r
P3.0_mdl <- glb_mdl
fit_df <- glb_entity_df; P4.0_mdl <- step(P1.1_mdl)
```

```
## Start:  AIC=-1348.16
## Temp ~ MEI + CO2 + CH4 + N2O + CFC.11 + CFC.12 + TSI + Aerosols
## 
##            Df Sum of Sq    RSS     AIC
## - CH4       1   0.00049 2.3135 -1350.1
## <none>                  2.3130 -1348.2
## - N2O       1   0.03132 2.3443 -1346.3
## - CO2       1   0.06719 2.3802 -1342.0
## - CFC.12    1   0.11874 2.4318 -1335.9
## - CFC.11    1   0.13986 2.4529 -1333.5
## - TSI       1   0.33516 2.6482 -1311.7
## - Aerosols  1   0.43727 2.7503 -1301.0
## - MEI       1   0.82823 3.1412 -1263.2
## 
## Step:  AIC=-1350.1
## Temp ~ MEI + CO2 + N2O + CFC.11 + CFC.12 + TSI + Aerosols
## 
##            Df Sum of Sq    RSS     AIC
## <none>                  2.3135 -1350.1
## - N2O       1   0.03133 2.3448 -1348.3
## - CO2       1   0.06672 2.3802 -1344.0
## - CFC.12    1   0.13023 2.4437 -1336.5
## - CFC.11    1   0.13938 2.4529 -1335.5
## - TSI       1   0.33500 2.6485 -1313.7
## - Aerosols  1   0.43987 2.7534 -1302.7
## - MEI       1   0.83118 3.1447 -1264.9
```

```r
glb_sel_mdl <- P4.0_mdl #glb_mdl

if (glb_is_regression)
    print(myplot_scatter(glb_models_df, "Adj.R.sq.fit", "R.sq.OOB") + 
          geom_text(aes(label=feats), data=glb_models_df, color="NavyBlue", 
                    size=3.5))
```

![](glb_Climate_files/figure-html/run_models-1.png) 

```r
if (glb_is_classification) {
    plot_models_df <- mutate(glb_models_df, feats.label=substr(feats, 1, 20))
    print(myplot_hbar(df=plot_models_df, xcol_name="feats.label", 
                      ycol_names="f.score.OOB"))
}

script_df <- rbind(script_df, 
                   data.frame(chunk_label="fit_training.all", 
                              chunk_step_major=max(script_df$chunk_step_major)+1, 
                              chunk_step_minor=0))
print(script_df)
```

```
##                   chunk_label chunk_step_major chunk_step_minor
## 1                 import_data                1                0
## 2                cleanse_data                2                0
## 3        inspect_explore_data                2                1
## 4         manage_missing_data                2                2
## 5          encode_retype_data                2                3
## 6            extract_features                3                0
## 7             select_features                4                0
## 8  remove_correlated_features                4                1
## 9                  run_models                5                0
## 10           fit_training.all                6                0
```

## Step `6`: fit training.all

```r
print(mdl_feats_df <- myextract_mdl_feats())
```

```
##              Estimate   Std. Error   t value         Pr.z       id
## MEI       0.064067786 0.0064338702  9.957892 3.710404e-20      MEI
## Aerosols -1.540205823 0.2126157579 -7.244081 4.358319e-12 Aerosols
## TSI       0.093115509 0.0147292923  6.321791 1.035487e-09      TSI
## CFC.11   -0.006609351 0.0016208317 -4.077753 5.953626e-05   CFC.11
## CFC.12    0.003867565 0.0009812172  3.941599 1.026435e-04   CFC.12
## CO2       0.006401495 0.0022689182  2.821387 5.128895e-03      CO2
## N2O      -0.016021128 0.0082873403 -1.933205 5.423364e-02      N2O
```

```r
if (glb_is_regression) {
    ret_lst <- myrun_mdl_lm(indep_vars_vctr=mdl_feats_df$id, fit_df=glb_entity_df)
    glb_sel_mdl <- glb_mdl    
    glb_entity_df[, glb_predct_var_name] <- predict(glb_sel_mdl, newdata=glb_entity_df)
    print(myplot_scatter(glb_entity_df, glb_predct_var, glb_predct_var_name, 
                         smooth=TRUE))    
}    
```

```
## 
## Call:
## lm(formula = reformulate(indep_vars_vctr, response = glb_predct_var), 
##     data = fit_df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.25770 -0.05994 -0.00104  0.05588  0.32203 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -1.245e+02  1.985e+01  -6.273 1.37e-09 ***
## MEI          6.407e-02  6.434e-03   9.958  < 2e-16 ***
## Aerosols    -1.540e+00  2.126e-01  -7.244 4.36e-12 ***
## TSI          9.312e-02  1.473e-02   6.322 1.04e-09 ***
## CFC.11      -6.609e-03  1.621e-03  -4.078 5.95e-05 ***
## CFC.12       3.868e-03  9.812e-04   3.942 0.000103 ***
## CO2          6.402e-03  2.269e-03   2.821 0.005129 ** 
## N2O         -1.602e-02  8.287e-03  -1.933 0.054234 .  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.09155 on 276 degrees of freedom
## Multiple R-squared:  0.7508,	Adjusted R-squared:  0.7445 
## F-statistic: 118.8 on 7 and 276 DF,  p-value: < 2.2e-16
## 
##                                               feats n.fit  R.sq.fit
## 4 MEI, CO2, CH4, N2O, CFC.11, CFC.12, TSI, Aerosols   284 0.7508933
## 5 MEI, CO2, CH4, N2O, CFC.11, CFC.12, TSI, Aerosols   284 0.7508933
## 6                           MEI, N2O, TSI, Aerosols   284 0.7261321
## 3                   CO2, CFC.11, TSI, MEI, Aerosols   284 0.7344547
## 2                 CO2, CO2:N2O, CO2:CH4, CO2:CFC.12   284 0.6237367
## 1                                               CO2   284 0.6217783
## 7      MEI, Aerosols, TSI, CFC.11, CFC.12, CO2, N2O   284 0.7508409
##     R.sq.OOB Adj.R.sq.fit  SSE.fit   SSE.OOB f.score.OOB
## 4  0.6274054    0.7508933 2.313020 0.2183475          NA
## 5  0.6274054    0.7508933 2.313020 0.2183475          NA
## 6  0.4967795    0.7261321 2.542933 0.2948967          NA
## 3  0.3596563    0.7344547 2.465656 0.3752535          NA
## 2 -0.6715997    0.6237367 3.493701 0.9795889          NA
## 1 -0.7919957    0.6217783 3.511885 1.0501433          NA
## 7         NA    0.7508409 2.313506        NA          NA
```

```
## geom_smooth: method="auto" and size of largest group is <1000, so using loess. Use 'method = x' to change the smoothing method.
```

![](glb_Climate_files/figure-html/fit_training.all-1.png) 

```r
if (glb_is_classification) {
    ret_lst <- myrun_mdl_glm(indep_vars_vctr=mdl_feats_df$id, fit_df=glb_entity_df)
    glb_sel_mdl <- glb_mdl        
    glb_entity_df[, glb_predct_var_name] <- (predict(glb_sel_mdl, 
                        newdata=glb_entity_df, type="response") >= 0.5) * 1.0
    print(xtabs(reformulate(paste(glb_predct_var, glb_predct_var_name, sep=" + ")),
                glb_entity_df))                        
}    

print(glb_feats_df <- mymerge_feats_Pr.z())
```

```
##         id      cor.y cor.y.abs cor.low         Pr.z
## 6      MEI  0.1724708 0.1724708       1 3.710404e-20
## 1 Aerosols -0.3849137 0.3849137       1 4.358319e-12
## 8      TSI  0.2433827 0.2433827       1 1.035487e-09
## 2   CFC.11  0.4077103 0.4077103       1 5.953626e-05
## 3   CFC.12  0.6875575 0.6875575      NA 1.026435e-04
## 5      CO2  0.7885292 0.7885292       1 5.128895e-03
## 7      N2O  0.7786389 0.7786389      NA 5.423364e-02
## 4      CH4  0.7032550 0.7032550      NA           NA
```

```r
# Most of this code is used again in predict_newdata chunk
glb_analytics_diag_plots <- function(obs_df) {
    for (var in subset(glb_feats_df, Pr.z < 0.1)$id) {
        plot_df <- melt(obs_df, id.vars=var, 
                        measure.vars=c(glb_predct_var, glb_predct_var_name))
#         if (var == "<feat_name>") print(myplot_scatter(plot_df, var, "value", 
#                                              facet_colcol_name="variable") + 
#                       geom_vline(xintercept=<divider_val>, linetype="dotted")) else     
            print(myplot_scatter(plot_df, var, "value", facet_colcol_name="variable"))
    }
    
    if (glb_is_regression) {
        plot_vars_df <- subset(glb_feats_df, Pr.z < 0.1)
        print(myplot_prediction_regression(obs_df, 
                    ifelse(nrow(plot_vars_df) > 1, plot_vars_df$id[2], ".rownames"), 
                                           plot_vars_df$id[1])
#               + geom_point(aes_string(color="<col_name>.fctr"))
              )
    }    
    
    if (glb_is_classification) {
        plot_vars_df <- subset(glb_feats_df, Pr.z < 0.1)
        print(myplot_prediction_classification(obs_df, 
                    ifelse(nrow(plot_vars_df) > 1, plot_vars_df$id[2], ".rownames"),
                                               plot_vars_df$id[1])
#               + geom_hline(yintercept=<divider_val>, linetype = "dotted")
                )
    }    
}
glb_analytics_diag_plots(glb_entity_df)
```

![](glb_Climate_files/figure-html/fit_training.all-2.png) ![](glb_Climate_files/figure-html/fit_training.all-3.png) ![](glb_Climate_files/figure-html/fit_training.all-4.png) ![](glb_Climate_files/figure-html/fit_training.all-5.png) ![](glb_Climate_files/figure-html/fit_training.all-6.png) ![](glb_Climate_files/figure-html/fit_training.all-7.png) ![](glb_Climate_files/figure-html/fit_training.all-8.png) 

```
##     Year Month    MEI    CO2     CH4     N2O  CFC.11  CFC.12      TSI
## 184 1998     8 -0.410 365.79 1757.06 313.619 265.784 536.489 1366.086
## 183 1998     7  0.285 367.74 1762.46 313.542 265.787 536.151 1366.244
## 190 1999     2 -1.238 368.98 1784.64 314.571 265.701 538.971 1366.239
## 178 1998     2  2.777 365.98 1766.01 313.512 266.978 535.561 1365.917
## 20  1984    12 -0.611 344.54 1672.15 305.313 207.308 372.701 1365.762
##     Aerosols   Temp Temp.predict Temp.predict.err  .label
## 184   0.0024  0.616   0.29397010        0.3220299  1998:8
## 183   0.0025  0.651   0.36548216        0.2855178  1998:7
## 190   0.0022  0.540   0.27083977        0.2691602  1999:2
## 178   0.0037  0.739   0.47182798        0.2671720  1998:2
## 20    0.0188 -0.282  -0.02429685        0.2577031 1984:12
```

![](glb_Climate_files/figure-html/fit_training.all-9.png) 

```r
script_df <- rbind(script_df, 
                   data.frame(chunk_label="predict_newdata", 
                              chunk_step_major=max(script_df$chunk_step_major)+1, 
                              chunk_step_minor=0))
print(script_df)
```

```
##                   chunk_label chunk_step_major chunk_step_minor
## 1                 import_data                1                0
## 2                cleanse_data                2                0
## 3        inspect_explore_data                2                1
## 4         manage_missing_data                2                2
## 5          encode_retype_data                2                3
## 6            extract_features                3                0
## 7             select_features                4                0
## 8  remove_correlated_features                4                1
## 9                  run_models                5                0
## 10           fit_training.all                6                0
## 11            predict_newdata                7                0
```

## Step `7`: predict newdata

```r
if (glb_is_regression)
    glb_predct_df[, glb_predct_var_name] <- predict(glb_sel_mdl, 
                                        newdata=glb_predct_df, type="response")

if (glb_is_classification)
    glb_predct_df[, glb_predct_var_name] <- (predict(glb_sel_mdl, 
                        newdata=glb_predct_df, type="response") >= 0.5) * 1.0
    
myprint_df(glb_predct_df[, c(glb_id_vars, glb_predct_var, glb_predct_var_name)])
```

```
##     Year Month  Temp Temp.predict
## 285 2007     1 0.601    0.4677808
## 286 2007     2 0.498    0.4435404
## 287 2007     3 0.435    0.4265541
## 288 2007     4 0.466    0.4299162
## 289 2007     5 0.372    0.4455113
## 290 2007     6 0.382    0.4151422
##     Year Month  Temp Temp.predict
## 285 2007     1 0.601    0.4677808
## 292 2007     8 0.358    0.3839390
## 293 2007     9 0.402    0.3255595
## 294 2007    10 0.362    0.3274147
## 302 2008     6 0.315    0.4391458
## 304 2008     8 0.407    0.3913679
##     Year Month  Temp Temp.predict
## 303 2008     7 0.406    0.4237965
## 304 2008     8 0.407    0.3913679
## 305 2008     9 0.378    0.3587615
## 306 2008    10 0.440    0.3451991
## 307 2008    11 0.394    0.3607087
## 308 2008    12 0.330    0.3638076
```

```r
if (glb_is_regression) {
    print(sprintf("Total SSE: %0.4f", 
                  sum((glb_predct_df[, glb_predct_var_name] - 
                        glb_predct_df[, glb_predct_var]) ^ 2)))
    print(myplot_scatter(glb_predct_df, glb_predct_var, glb_predct_var_name, 
                         smooth=TRUE))
}                         
```

```
## [1] "Total SSE: 0.2176"
```

```
## geom_smooth: method="auto" and size of largest group is <1000, so using loess. Use 'method = x' to change the smoothing method.
```

![](glb_Climate_files/figure-html/predict_newdata-1.png) 

```r
if (glb_is_classification)
    print(xtabs(reformulate(paste(glb_predct_var, glb_predct_var_name, sep=" + ")),
                glb_predct_df))
    
glb_analytics_diag_plots(glb_predct_df)
```

![](glb_Climate_files/figure-html/predict_newdata-2.png) ![](glb_Climate_files/figure-html/predict_newdata-3.png) ![](glb_Climate_files/figure-html/predict_newdata-4.png) ![](glb_Climate_files/figure-html/predict_newdata-5.png) ![](glb_Climate_files/figure-html/predict_newdata-6.png) ![](glb_Climate_files/figure-html/predict_newdata-7.png) ![](glb_Climate_files/figure-html/predict_newdata-8.png) 

```
##     Year Month    MEI    CO2     CH4     N2O  CFC.11  CFC.12      TSI
## 297 2008     1 -1.011 385.44 1809.92 321.328 246.183 536.876 1365.716
## 298 2008     2 -1.402 385.73 1803.45 321.345 245.898 536.484 1365.737
## 301 2008     5 -0.355 388.50 1796.43 321.420 244.914 535.399 1365.717
## 285 2007     1  0.974 382.93 1799.66 320.561 248.372 539.206 1365.717
## 299 2008     3 -1.635 385.97 1792.84 321.295 245.430 535.979 1365.673
##     Aerosols  Temp Temp.predict Temp.predict.err .label
## 297   0.0038 0.074    0.3522134        0.2782134 2008:1
## 298   0.0036 0.198    0.3313129        0.1333129 2008:2
## 301   0.0031 0.283    0.4162213        0.1332213 2008:5
## 285   0.0054 0.601    0.4677808        0.1332192 2007:1
## 299   0.0034 0.447    0.3142112        0.1327888 2008:3
```

![](glb_Climate_files/figure-html/predict_newdata-9.png) 

Null Hypothesis ($\sf{H_{0}}$): mpg is not impacted by am_fctr.  
The variance by am_fctr appears to be independent. 

```r
# print(t.test(subset(cars_df, am_fctr == "automatic")$mpg, 
#              subset(cars_df, am_fctr == "manual")$mpg, 
#              var.equal=FALSE)$conf)
```
We reject the null hypothesis i.e. we have evidence to conclude that am_fctr impacts mpg (95% confidence). Manual transmission is better for miles per gallon versus automatic transmission.


```
## R version 3.1.2 (2014-10-31)
## Platform: x86_64-apple-darwin13.4.0 (64-bit)
## 
## locale:
## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
## [1] reshape2_1.4.1  plyr_1.8.1      doBy_4.5-13     survival_2.38-1
## [5] ggplot2_1.0.0  
## 
## loaded via a namespace (and not attached):
##  [1] colorspace_1.2-5 digest_0.6.8     evaluate_0.5.5   formatR_1.0     
##  [5] grid_3.1.2       gtable_0.1.2     htmltools_0.2.6  knitr_1.9       
##  [9] labeling_0.3     lattice_0.20-30  MASS_7.3-39      Matrix_1.1-5    
## [13] munsell_0.4.2    proto_0.3-10     Rcpp_0.11.4      rmarkdown_0.5.1 
## [17] scales_0.2.4     splines_3.1.2    stringr_0.6.2    tcltk_3.1.2     
## [21] tools_3.1.2      yaml_2.1.13
```
