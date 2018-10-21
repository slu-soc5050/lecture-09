Lecture 09 Examples
================
Christopher Prener, Ph.D.
(October 21, 2018)

## Introduction

This notebook introduces techniques for working with factor variables in
`R`.

## Dependencies

This notebook requires the `dplyr` and `forcats` packages for working
with data.

``` r
# tidyverse packages
library(dplyr)       # wrangling data
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(forcats)     # wrangling factors

# other packages
library(janitor)     # frequency tables
library(testDriveR)  # sample data
```

## Load Data

This notebook requires the `gss14_simple` data from `testDriveR`:

``` r
gss <- gss14_simple
```

These data are missing some important features, like explicitly
identified missing data and descriptive value labels. We’ll address both
today as part of Lecture 09.

## Missing Data

First, we’ll discuss how to manage data that should be missing. Take for
example the variable `SPDEG`, which contains the highest degree earned
by the spouse of each respondent. This is what the `SPDEG` variable
looks like in the `gss` data frame:

``` r
gss %>%
  tabyl(SPDEG)
```

    ##  SPDEG    n     percent
    ##     -1  210 0.082742317
    ##      0  129 0.050827423
    ##      1  529 0.208431836
    ##      2   89 0.035066982
    ##      3  238 0.093774626
    ##      4  169 0.066587864
    ##      7 1166 0.459416864
    ##      8    4 0.001576044
    ##      9    4 0.001576044

We see a variety of values, including `-1`. This should immediately
stand out because it is negative in a variable where all other values
are positive. Using a negative value is a common way to represent
missing data explicitly. The other two values that pop out on this table
are `8` and `9`, because they are both very uncommon in the data. If we
go to the [GSS
codebook](https://gssdataexplorer.norc.org/variables/62/vshow), we can
see that `-1` is used as a missing data indicator, `7` is “Not
applicable”, `8` is “Don’t know”, and `9` is “No answer”.

## Missing with `ifelse()`

We can declare these all explicitly missing in one of two ways. The
first works well if you have a single value to declare missing. We’ll
use `ifelse()` inside a `mutate()` call, just like we have done to make
other recodes of individual values.

``` r
gss <- mutate(gss, SPDEG = ifelse(SPDEG == -1, NA, SPDEG))

gss %>%
  tabyl(SPDEG)
```

    ##  SPDEG    n     percent valid_percent
    ##      0  129 0.050827423   0.055412371
    ##      1  529 0.208431836   0.227233677
    ##      2   89 0.035066982   0.038230241
    ##      3  238 0.093774626   0.102233677
    ##      4  169 0.066587864   0.072594502
    ##      7 1166 0.459416864   0.500859107
    ##      8    4 0.001576044   0.001718213
    ##      9    4 0.001576044   0.001718213
    ##     NA  210 0.082742317            NA

We instruct `mutate()` to replace values of `-1` with the value `NA`,
all other values are left alone (by including `SPDEG`’s current values
as the `FALSE` outcome in `ifelse()`). Notice now that the frequency
table shows those 210 `-1` values as `NA` (or “missing”) values instead,
and a column for valid percent has been added.

## Recoding Valid and Missing Data with `case_when()`

If we have multiple missing categories, we can use `case_when()`
instead. This is a very helpful function for working with discrete
variables that are either numeric or character/string. This time, we’ll
declare `7`, `8`, and `9` as missing since they represent non-concrete
answers from respondents. We’ll also add value labels to the first five
valid values.

``` r
gss <- mutate(gss, SPDEG = case_when(
  SPDEG == 0 ~ "Less than high school (0)",
  SPDEG == 1 ~ "High school (1)",
  SPDEG == 2 ~ "Junior college (2)",
  SPDEG == 3 ~ "Bachelor (3)",
  SPDEG == 4 ~ "Graduate (4)",
  SPDEG == 7 ~ NA_character_,
  SPDEG == 8 ~ NA_character_,
  SPDEG == 9 ~ NA_character_))

gss %>%
  tabyl(SPDEG) %>%
  adorn_pct_formatting(digits = 2)
```

    ##                      SPDEG    n percent valid_percent
    ##               Bachelor (3)  238   9.38%        20.62%
    ##               Graduate (4)  169   6.66%        14.64%
    ##            High school (1)  529  20.84%        45.84%
    ##         Junior college (2)   89   3.51%         7.71%
    ##  Less than high school (0)  129   5.08%        11.18%
    ##                       <NA> 1384  54.53%             -

Notice a couple of things:

1.  We’ve converted the `SPDEG` data to character to add descriptive
    labels.
2.  We’ve included the original numeric value paranthetically in each
    character value.
3.  Missingness was declared using `NA_character_`. In `case_when()`,
    missing values are declared like so:
      - `NA` - logical variables
      - `NA_character_` - character variables
      - `NA_integer_` - integer variables
      - `NA_real_` - all other numeric variables
4.  Within `case_when()` a tilde (`~`) is used to separate logical tests
    from the new value.
5.  Each logical test is separated by a comma, and all possible values
    must be explicitly identified.
6.  We can use `adorn_pct_formatting(digits = 2)` to round the
    percentage values to two significant digits, and to diplay them as
    percentages instead of proportions.

Notice as well that the values are not presented in numerical order but
rather in alphabetical order. This the downside to storing categorical
variables as strings - their specified ordering gets lost in `R`.
Instead, we want to store these as factors.

## Reodering Factors

If we want to reoder our string variable as a factor, we need to convert
it to factor and then use the `forcats::fct_relevel()` function to
reorder our levels:

``` r
gss <- mutate(gss, SPDEG = fct_relevel(as.factor(SPDEG), 
                                       "Less than high school (0)",
                                       "High school (1)",
                                       "Junior college (2)",
                                       "Bachelor (3)",
                                       "Graduate (4)"))

gss %>%
  tabyl(SPDEG) %>%
  adorn_pct_formatting(digits = 2)
```

    ##                      SPDEG    n percent valid_percent
    ##  Less than high school (0)  129   5.08%        11.18%
    ##            High school (1)  529  20.84%        45.84%
    ##         Junior college (2)   89   3.51%         7.71%
    ##               Bachelor (3)  238   9.38%        20.62%
    ##               Graduate (4)  169   6.66%        14.64%
    ##                       <NA> 1384  54.53%             -

Now, our `SPDEG` variable is converted to a factor. The frequency table
shows that the levels have been reordered as we wished. The
`fct_relevel()` function will follow the order you list the labels in
when the function is called.

## Creating Factors

This workflow is complicated, however. We declared values as missing,
then created a string, then created a factor. We can simplify this with
a pipeline using `mutate()` and `fct_recode()`. Our pipeline first
declares both `7` and `8` as missing using `ifelse()` wrapped in
`mutate()`. Then, we use `fct_recode()` wrapped in `mutate()` to add
labels to each level of the factor.

``` r
gss %>%
  mutate(MADEG = ifelse(MADEG == 7 | MADEG == 8, NA, MADEG)) %>%
  mutate(MADEG = fct_recode(as.factor(MADEG),
                            "Less than high school (0)" = "0",
                            "High school (1)" = "1",
                            "Junior college (2)" = "2",
                            "Bachelor (3)" = "3",
                            "Graduate (4)" = "4")) -> gss

gss %>%
  tabyl(MADEG) %>%
  adorn_pct_formatting(digits = 2)
```

    ##                      MADEG    n percent valid_percent
    ##  Less than high school (0)  698  27.50%        29.63%
    ##            High school (1) 1160  45.71%        49.24%
    ##         Junior college (2)  139   5.48%         5.90%
    ##               Bachelor (3)  228   8.98%         9.68%
    ##               Graduate (4)  131   5.16%         5.56%
    ##                       <NA>  182   7.17%             -

## Modifying Factors

We can also use `fct_recode()` to modify levels. Given the previous
`MADEG` variable, imagine we wanted to limit it to three outcomes -
`Less than high school`, `High school`, and `More than high school`. We
can use `fct_recode()` again to accomplish this, creating a new variable
called `MADEG_simple`:

``` r
gss <- mutate(gss, MADEG_simple = fct_recode(MADEG,
                                      "Less than high school" = "Less than high school (0)",
                                      "High school" = "High school (1)",
                                      "More than high school" = "Junior college (2)",
                                      "More than high school" = "Bachelor (3)",
                                      "More than high school" = "Graduate (4)"))

gss %>%
  tabyl(MADEG_simple) %>%
  adorn_pct_formatting(digits = 2)
```

    ##           MADEG_simple    n percent valid_percent
    ##  Less than high school  698  27.50%        29.63%
    ##            High school 1160  45.71%        49.24%
    ##  More than high school  498  19.62%        21.14%
    ##                   <NA>  182   7.17%             -

`fct_recode()` will follow the order of the levels specified, and will
collapse values into the new categories to the left of each equals sign.

## When To Use Functions

  - `ifelse()` - use when you have a small number of edits to make to
    numeric variables (in terms of declaring values missing) and string
    variables (for declaring values missing or editing a single specific
    value)
  - `case_when()` - use for categorical variables to convert to string
    and/or declare a range of values missing
  - `fct_relevel()` - use for editing the order of *already existing*
    factors
  - `fct_recode()` - use for specifying values for *new* factors, or for
    changing the values of *existing* factors
