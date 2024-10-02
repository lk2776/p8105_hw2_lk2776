problem3
================
2024-10-02

bakers data is read using read_csv() from tidyverse, column names are
cleaned by janitor::clean_names(). ‘baker’ variable is created from
first part of baker_name variable using separate(), and the ‘rest_of_the
name’ is removed from the dataset. This additional variable is created
to join it with bakes_data, which has only baker variable (not the full
name) bakes data is read in a similar fashion, joined with bakers_data
with full_join(). Similarly, results_data is read using read_csv(),
skipped first 2 lines of the dataset, and joined with bakers_bakes_data
with full_join(). anti_join() is used to verify that the data is not
missed while joining the tables.

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.3     ✔ tidyr     1.3.1
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
bakers_data = read_csv("./data/bakers.csv", 
                       col_names=TRUE,
                       na=c("NA",".","")) |>
  janitor::clean_names() |>
  separate(baker_name,into=c("baker","rest_of_the_name"), sep=" ", remove=FALSE) |>
  select(-rest_of_the_name)
```

    ## Rows: 120 Columns: 5
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (3): Baker Name, Baker Occupation, Hometown
    ## dbl (2): Series, Baker Age
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
bakes_data = read_csv("./data/bakes.csv", 
                      col_names=TRUE,
                      na=c("NA",".","")) |> 
  janitor::clean_names() 
```

    ## Rows: 548 Columns: 5
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (3): Baker, Signature Bake, Show Stopper
    ## dbl (2): Series, Episode
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
#join the datasets by full_join
bakers_bakes_df = 
  full_join(bakers_data, bakes_data, by = c("baker", "series")) |>
  relocate(baker_name,baker,baker_age,baker_occupation,hometown,series,episode,signature_bake,show_stopper)

#check for completeness and correctness of data
anti_join(bakers_data, bakes_data, by = c("baker", "series"))
```

    ## # A tibble: 26 × 6
    ##    baker_name          baker  series baker_age baker_occupation         hometown
    ##    <chr>               <chr>   <dbl>     <dbl> <chr>                    <chr>   
    ##  1 Alice Fevronia      Alice      10        28 Geography teacher        Essex   
    ##  2 Amelia LeBruin      Amelia     10        24 Fashion designer         Halifax 
    ##  3 Antony Amourdoux    Antony      9        30 Banker                   London  
    ##  4 Briony Williams     Briony      9        33 Full-time parent         Bristol 
    ##  5 Dan Beasley-Harling Dan         9        36 Full-time parent         London  
    ##  6 Dan Chambers        Dan        10        32 Support worker           Rotherh…
    ##  7 David Atherton      David      10        36 International health ad… Whitby  
    ##  8 Helena Garcia       Helena     10        40 Online project manager   Leeds   
    ##  9 Henry Bird          Henry      10        20 Student                  Durham  
    ## 10 Imelda McCarron     Imelda      9        33 Countryside recreation … County …
    ## # ℹ 16 more rows

``` r
anti_join(bakes_data, bakers_data, by = c("baker", "series"))
```

    ## # A tibble: 8 × 5
    ##   series episode baker    signature_bake                            show_stopper
    ##    <dbl>   <dbl> <chr>    <chr>                                     <chr>       
    ## 1      2       1 "\"Jo\"" Chocolate Orange CupcakesOrange and Card… Chocolate a…
    ## 2      2       2 "\"Jo\"" Caramelised Onion, Gruyere and Thyme Qui… Raspberry a…
    ## 3      2       3 "\"Jo\"" Stromboli flavored with Mozzarella, Ham,… Unknown     
    ## 4      2       4 "\"Jo\"" Lavender Biscuits                         Blueberry M…
    ## 5      2       5 "\"Jo\"" Salmon and Asparagus Pie                  Apple and R…
    ## 6      2       6 "\"Jo\"" Rum and Raisin Baked Cheesecake           Limoncello …
    ## 7      2       7 "\"Jo\"" Raspberry & Strawberry Mousse Cake        Pain Aux Ra…
    ## 8      2       8 "\"Jo\"" Raspberry and Blueberry Mille Feuille     Mini Victor…

``` r
#read results data 
results_data = read_csv("./data/results.csv", 
                        col_names=TRUE,
                        na=c("NA",".",""), skip=2) |>
  janitor::clean_names() 
```

    ## Rows: 1136 Columns: 5
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): baker, result
    ## dbl (3): series, episode, technical
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
#join results data with bakers_bakes_df
bakers_bakes_results_df = 
full_join(bakers_bakes_df, results_data, by=c("baker","series","episode"))
write_csv(bakers_bakes_results_df,file="./data/bakers_bakes_results_df.csv")

#check for completeness of data
anti_join(bakers_bakes_df, results_data, by = c("baker","series","episode"))
```

    ## # A tibble: 34 × 9
    ##    baker_name          baker  baker_age baker_occupation hometown series episode
    ##    <chr>               <chr>      <dbl> <chr>            <chr>     <dbl>   <dbl>
    ##  1 Alice Fevronia      Alice         28 Geography teach… Essex        10      NA
    ##  2 Amelia LeBruin      Amelia        24 Fashion designer Halifax      10      NA
    ##  3 Antony Amourdoux    Antony        30 Banker           London        9      NA
    ##  4 Briony Williams     Briony        33 Full-time parent Bristol       9      NA
    ##  5 Dan Beasley-Harling Dan           36 Full-time parent London        9      NA
    ##  6 Dan Chambers        Dan           32 Support worker   Rotherh…     10      NA
    ##  7 David Atherton      David         36 International h… Whitby       10      NA
    ##  8 Helena Garcia       Helena        40 Online project … Leeds        10      NA
    ##  9 Henry Bird          Henry         20 Student          Durham       10      NA
    ## 10 Imelda McCarron     Imelda        33 Countryside rec… County …      9      NA
    ## # ℹ 24 more rows
    ## # ℹ 2 more variables: signature_bake <chr>, show_stopper <chr>

``` r
anti_join(results_data, bakers_bakes_df,by = c("baker","series","episode"))
```

    ## # A tibble: 596 × 5
    ##    series episode baker    technical result
    ##     <dbl>   <dbl> <chr>        <dbl> <chr> 
    ##  1      1       2 Lea             NA <NA>  
    ##  2      1       2 Mark            NA <NA>  
    ##  3      1       3 Annetha         NA <NA>  
    ##  4      1       3 Lea             NA <NA>  
    ##  5      1       3 Louise          NA <NA>  
    ##  6      1       3 Mark            NA <NA>  
    ##  7      1       4 Annetha         NA <NA>  
    ##  8      1       4 Jonathan        NA <NA>  
    ##  9      1       4 Lea             NA <NA>  
    ## 10      1       4 Louise          NA <NA>  
    ## # ℹ 586 more rows

The no. of rows and columns of final dataset are: **1170, 11** The
variables of the final datasets are: **baker_name, baker, baker_age,
baker_occupation, hometown, series, episode, signature_bake,
show_stopper, technical, result**

``` r
#create a table for starbaker/winner for each episode in season 5 to 10
table = bakers_bakes_results_df |>  filter(result==c("WINNER","STAR BAKER")) |>
  filter(series >= 5) |>
  arrange(series,episode)
```

``` r
#read viewers_df
viewers_df = read_csv("./data/viewers.csv", 
                        col_names=TRUE,
                        na=c("NA",".","")) |>
  janitor::clean_names() 
```

    ## Rows: 10 Columns: 11
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## dbl (11): Episode, Series 1, Series 2, Series 3, Series 4, Series 5, Series ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
#print viewers data
print(viewers_df)
```

    ## # A tibble: 10 × 11
    ##    episode series_1 series_2 series_3 series_4 series_5 series_6 series_7
    ##      <dbl>    <dbl>    <dbl>    <dbl>    <dbl>    <dbl>    <dbl>    <dbl>
    ##  1       1     2.24     3.1      3.85     6.6      8.51     11.6     13.6
    ##  2       2     3        3.53     4.6      6.65     8.79     11.6     13.4
    ##  3       3     3        3.82     4.53     7.17     9.28     12.0     13.0
    ##  4       4     2.6      3.6      4.71     6.82    10.2      12.4     13.3
    ##  5       5     3.03     3.83     4.61     6.95     9.95     12.4     13.1
    ##  6       6     2.75     4.25     4.82     7.32    10.1      12       13.1
    ##  7       7    NA        4.42     5.1      7.76    10.3      12.4     13.4
    ##  8       8    NA        5.06     5.35     7.41     9.02     11.1     13.3
    ##  9       9    NA       NA        5.7      7.41    10.7      12.6     13.4
    ## 10      10    NA       NA        6.74     9.45    13.5      15.0     15.9
    ## # ℹ 3 more variables: series_8 <dbl>, series_9 <dbl>, series_10 <dbl>

``` r
round(mean(pull(viewers_df, series_1), na.rm=TRUE), digits=2)
```

    ## [1] 2.77

``` r
round(mean(pull(viewers_df, series_5), na.rm=TRUE), digits=2)
```

    ## [1] 10.04
