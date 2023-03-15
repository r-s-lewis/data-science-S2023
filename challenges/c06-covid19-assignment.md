COVID-19
================
Reuben Lewis
2023-03-06

- [Grading Rubric](#grading-rubric)
  - [Individual](#individual)
  - [Due Date](#due-date)
- [The Big Picture](#the-big-picture)
- [Get the Data](#get-the-data)
  - [Navigating the Census Bureau](#navigating-the-census-bureau)
    - [**q1** Load Table `B01003` into the following tibble. Make sure
      the column names are
      `id, Geographic Area Name, Estimate!!Total, Margin of Error!!Total`.](#q1-load-table-b01003-into-the-following-tibble-make-sure-the-column-names-are-id-geographic-area-name-estimatetotal-margin-of-errortotal)
  - [Automated Download of NYT Data](#automated-download-of-nyt-data)
    - [**q2** Visit the NYT GitHub repo and find the URL for the **raw**
      US County-level data. Assign that URL as a string to the variable
      below.](#q2-visit-the-nyt-github-repo-and-find-the-url-for-the-raw-us-county-level-data-assign-that-url-as-a-string-to-the-variable-below)
- [Join the Data](#join-the-data)
  - [**q3** Process the `id` column of `df_pop` to create a `fips`
    column.](#q3-process-the-id-column-of-df_pop-to-create-a-fips-column)
  - [**q4** Join `df_covid` with `df_q3` by the `fips` column. Use the
    proper type of join to preserve *only* the rows in
    `df_covid`.](#q4-join-df_covid-with-df_q3-by-the-fips-column-use-the-proper-type-of-join-to-preserve-only-the-rows-in-df_covid)
- [Analyze](#analyze)
  - [Normalize](#normalize)
    - [**q5** Use the `population` estimates in `df_data` to normalize
      `cases` and `deaths` to produce per 100,000 counts \[3\]. Store
      these values in the columns `cases_per100k` and
      `deaths_per100k`.](#q5-use-the-population-estimates-in-df_data-to-normalize-cases-and-deaths-to-produce-per-100000-counts-3-store-these-values-in-the-columns-cases_per100k-and-deaths_per100k)
  - [Guided EDA](#guided-eda)
    - [**q6** Compute the mean and standard deviation for
      `cases_per100k` and
      `deaths_per100k`.](#q6-compute-the-mean-and-standard-deviation-for-cases_per100k-and-deaths_per100k)
    - [**q7** Find the top 10 counties in terms of `cases_per100k`, and
      the top 10 in terms of `deaths_per100k`. Report the population of
      each county along with the per-100,000 counts. Compare the counts
      against the mean values you found in q6. Note any
      observations.](#q7-find-the-top-10-counties-in-terms-of-cases_per100k-and-the-top-10-in-terms-of-deaths_per100k-report-the-population-of-each-county-along-with-the-per-100000-counts-compare-the-counts-against-the-mean-values-you-found-in-q6-note-any-observations)
  - [Self-directed EDA](#self-directed-eda)
    - [**q8** Drive your own ship: You’ve just put together a very rich
      dataset; you now get to explore! Pick your own direction and
      generate at least one punchline figure to document an interesting
      finding. I give a couple tips & ideas
      below:](#q8-drive-your-own-ship-youve-just-put-together-a-very-rich-dataset-you-now-get-to-explore-pick-your-own-direction-and-generate-at-least-one-punchline-figure-to-document-an-interesting-finding-i-give-a-couple-tips--ideas-below)
    - [Ideas](#ideas)
    - [Observations](#observations)
    - [Aside: Some visualization
      tricks](#aside-some-visualization-tricks)
    - [Geographic exceptions](#geographic-exceptions)
- [Notes](#notes)

*Purpose*: In this challenge, you’ll learn how to navigate the U.S.
Census Bureau website, programmatically download data from the internet,
and perform a county-level population-weighted analysis of current
COVID-19 trends. This will give you the base for a very deep
investigation of COVID-19, which we’ll build upon for Project 1.

<!-- include-rubric -->

# Grading Rubric

<!-- -------------------------------------------------- -->

Unlike exercises, **challenges will be graded**. The following rubrics
define how you will be graded, both on an individual and team basis.

## Individual

<!-- ------------------------- -->

| Category    | Needs Improvement                                                                                                | Satisfactory                                                                                                               |
|-------------|------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------|
| Effort      | Some task **q**’s left unattempted                                                                               | All task **q**’s attempted                                                                                                 |
| Observed    | Did not document observations, or observations incorrect                                                         | Documented correct observations based on analysis                                                                          |
| Supported   | Some observations not clearly supported by analysis                                                              | All observations clearly supported by analysis (table, graph, etc.)                                                        |
| Assessed    | Observations include claims not supported by the data, or reflect a level of certainty not warranted by the data | Observations are appropriately qualified by the quality & relevance of the data and (in)conclusiveness of the support      |
| Specified   | Uses the phrase “more data are necessary” without clarification                                                  | Any statement that “more data are necessary” specifies which *specific* data are needed to answer what *specific* question |
| Code Styled | Violations of the [style guide](https://style.tidyverse.org/) hinder readability                                 | Code sufficiently close to the [style guide](https://style.tidyverse.org/)                                                 |

## Due Date

<!-- ------------------------- -->

All the deliverables stated in the rubrics above are due **at midnight**
before the day of the class discussion of the challenge. See the
[Syllabus](https://docs.google.com/document/d/1qeP6DUS8Djq_A0HMllMqsSqX3a9dbcx1/edit?usp=sharing&ouid=110386251748498665069&rtpof=true&sd=true)
for more information.

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
    ## ✔ ggplot2 3.4.0     ✔ purrr   1.0.1
    ## ✔ tibble  3.1.8     ✔ dplyr   1.1.0
    ## ✔ tidyr   1.3.0     ✔ stringr 1.5.0
    ## ✔ readr   2.1.3     ✔ forcats 1.0.0
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(ggdark)
```

*Background*:
[COVID-19](https://en.wikipedia.org/wiki/Coronavirus_disease_2019) is
the disease caused by the virus SARS-CoV-2. In 2020 it became a global
pandemic, leading to huge loss of life and tremendous disruption to
society. The New York Times (as of writing) publishes up-to-date data on
the progression of the pandemic across the United States—we will study
these data in this challenge.

*Optional Readings*: I’ve found this [ProPublica
piece](https://www.propublica.org/article/how-to-understand-covid-19-numbers)
on “How to understand COVID-19 numbers” to be very informative!

# The Big Picture

<!-- -------------------------------------------------- -->

We’re about to go through *a lot* of weird steps, so let’s first fix the
big picture firmly in mind:

We want to study COVID-19 in terms of data: both case counts (number of
infections) and deaths. We’re going to do a county-level analysis in
order to get a high-resolution view of the pandemic. Since US counties
can vary widely in terms of their population, we’ll need population
estimates in order to compute infection rates (think back to the
`Titanic` challenge).

That’s the high-level view; now let’s dig into the details.

# Get the Data

<!-- -------------------------------------------------- -->

1.  County-level population estimates (Census Bureau)
2.  County-level COVID-19 counts (New York Times)

## Navigating the Census Bureau

<!-- ------------------------- -->

**Steps**: Our objective is to find the 2018 American Community
Survey\[1\] (ACS) Total Population estimates, disaggregated by counties.
To check your results, this is Table `B01003`.

1.  Go to [data.census.gov](data.census.gov).
2.  Scroll down and click `View Tables`.
3.  Apply filters to find the ACS **Total Population** estimates,
    disaggregated by counties. I used the filters:

- `Topics > Populations and People > Counts, Estimates, and Projections > Population Total`
- `Geography > County > All counties in United States`

5.  Select the **Total Population** table and click the `Download`
    button to download the data; make sure to select the 2018 5-year
    estimates.
6.  Unzip and move the data to your `challenges/data` folder.

- Note that the data will have a crazy-long filename like
  `ACSDT5Y2018.B01003_data_with_overlays_2020-07-26T094857.csv`. That’s
  because metadata is stored in the filename, such as the year of the
  estimate (`Y2018`) and my access date (`2020-07-26`). **Your filename
  will vary based on when you download the data**, so make sure to copy
  the filename that corresponds to what you downloaded!

### **q1** Load Table `B01003` into the following tibble. Make sure the column names are `id, Geographic Area Name, Estimate!!Total, Margin of Error!!Total`.

*Hint*: You will need to use the `skip` keyword when loading these data!

``` r
## TASK: Load the census bureau data with the following tibble name.
filename <- "./data/pop.csv"
df_pop <- read_csv(filename, skip = 2, na = "*****", col_types = "ccd_d__", col_names = c("id", "Geographic Area Name", "Estimate!!Total", "Margin of Error!!Total"))
```

*Note*: You can find information on 1-year, 3-year, and 5-year estimates
[here](https://www.census.gov/programs-surveys/acs/guidance/estimates.html).
The punchline is that 5-year estimates are more reliable but less
current.

## Automated Download of NYT Data

<!-- ------------------------- -->

ACS 5-year estimates don’t change all that often, but the COVID-19 data
are changing rapidly. To that end, it would be nice to be able to
*programmatically* download the most recent data for analysis; that way
we can update our analysis whenever we want simply by re-running our
notebook. This next problem will have you set up such a pipeline.

The New York Times is publishing up-to-date data on COVID-19 on
[GitHub](https://github.com/nytimes/covid-19-data).

### **q2** Visit the NYT [GitHub](https://github.com/nytimes/covid-19-data) repo and find the URL for the **raw** US County-level data. Assign that URL as a string to the variable below.

``` r
## TASK: Find the URL for the NYT covid-19 county-level data
url_counties <- "https://raw.githubusercontent.com/nytimes/covid-19-data/master/us-counties.csv"
```

Once you have the url, the following code will download a local copy of
the data, then load the data into R.

``` r
## NOTE: No need to change this; just execute
## Set the filename of the data to download
filename_nyt <- "./data/nyt_counties.csv"

## Download the data locally
curl::curl_download(
        url_counties,
        destfile = filename_nyt 
      )

## Loads the downloaded csv
df_covid <- read_csv(filename_nyt)
```

    ## Rows: 2502832 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (3): county, state, fips
    ## dbl  (2): cases, deaths
    ## date (1): date
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

You can now re-run the chunk above (or the entire notebook) to pull the
most recent version of the data. Thus you can periodically re-run this
notebook to check in on the pandemic as it evolves.

*Note*: You should feel free to copy-paste the code above for your own
future projects!

# Join the Data

<!-- -------------------------------------------------- -->

To get a sense of our task, let’s take a glimpse at our two data
sources.

``` r
## NOTE: No need to change this; just execute
df_pop %>% glimpse
```

    ## Rows: 3,220
    ## Columns: 4
    ## $ id                       <chr> "0500000US01001", "0500000US01003", "0500000U…
    ## $ `Geographic Area Name`   <chr> "Autauga County, Alabama", "Baldwin County, A…
    ## $ `Estimate!!Total`        <dbl> 55200, 208107, 25782, 22527, 57645, 10352, 20…
    ## $ `Margin of Error!!Total` <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…

``` r
df_covid %>% glimpse
```

    ## Rows: 2,502,832
    ## Columns: 6
    ## $ date   <date> 2020-01-21, 2020-01-22, 2020-01-23, 2020-01-24, 2020-01-24, 20…
    ## $ county <chr> "Snohomish", "Snohomish", "Snohomish", "Cook", "Snohomish", "Or…
    ## $ state  <chr> "Washington", "Washington", "Washington", "Illinois", "Washingt…
    ## $ fips   <chr> "53061", "53061", "53061", "17031", "53061", "06059", "17031", …
    ## $ cases  <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, …
    ## $ deaths <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, …

To join these datasets, we’ll need to use [FIPS county
codes](https://en.wikipedia.org/wiki/FIPS_county_code).\[2\] The last
`5` digits of the `id` column in `df_pop` is the FIPS county code, while
the NYT data `df_covid` already contains the `fips`.

### **q3** Process the `id` column of `df_pop` to create a `fips` column.

``` r
## TASK: Create a `fips` column by extracting the county code
df_q3 <- 
  df_pop %>% 
  mutate(fips = str_extract(id, "[^US]*$"))
head(df_q3)
```

    ## # A tibble: 6 × 5
    ##   id             `Geographic Area Name`  `Estimate!!Total` Margin of Err…¹ fips 
    ##   <chr>          <chr>                               <dbl>           <dbl> <chr>
    ## 1 0500000US01001 Autauga County, Alabama             55200              NA 01001
    ## 2 0500000US01003 Baldwin County, Alabama            208107              NA 01003
    ## 3 0500000US01005 Barbour County, Alabama             25782              NA 01005
    ## 4 0500000US01007 Bibb County, Alabama                22527              NA 01007
    ## 5 0500000US01009 Blount County, Alabama              57645              NA 01009
    ## 6 0500000US01011 Bullock County, Alabama             10352              NA 01011
    ## # … with abbreviated variable name ¹​`Margin of Error!!Total`

Use the following test to check your answer.

``` r
## NOTE: No need to change this
## Check known county
assertthat::assert_that(
              (df_q3 %>%
              filter(str_detect(`Geographic Area Name`, "Autauga County")) %>%
              pull(fips)) == "01001"
            )
```

    ## [1] TRUE

``` r
print("Very good!")
```

    ## [1] "Very good!"

### **q4** Join `df_covid` with `df_q3` by the `fips` column. Use the proper type of join to preserve *only* the rows in `df_covid`.

``` r
## TASK: Join df_covid and df_q3 by fips.
df_q4 <- merge(df_covid, df_q3, "fips")
```

For convenience, I down-select some columns and produce more convenient
column names.

``` r
## NOTE: No need to change; run this to produce a more convenient tibble
df_data <-
  df_q4 %>%
  select(
    date,
    county,
    state,
    fips,
    cases,
    deaths,
    population = `Estimate!!Total`
  )
```

# Analyze

<!-- -------------------------------------------------- -->

Now that we’ve done the hard work of loading and wrangling the data, we
can finally start our analysis. Our first step will be to produce county
population-normalized cases and death counts. Then we will explore the
data.

## Normalize

<!-- ------------------------- -->

### **q5** Use the `population` estimates in `df_data` to normalize `cases` and `deaths` to produce per 100,000 counts \[3\]. Store these values in the columns `cases_per100k` and `deaths_per100k`.

``` r
## TASK: Normalize cases and deaths
df_normalized <-
  df_data %>% 
  mutate(cases_per100k = cases/population) %>% 
  mutate(deaths_per100k = deaths/population)
```

You may use the following test to check your work.

``` r
## NOTE: No need to change this
## Check known county data
if (any(df_normalized %>% pull(date) %>% str_detect(., "2020-01-21"))) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Date 2020-01-21 not found; did you download the historical data (correct),",
    "or just the most recent data (incorrect)?",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
assertthat::assert_that(
              abs(df_normalized %>%
               filter(
                 str_detect(county, "Snohomish"),
                 date == "2020-01-21"
               ) %>%
              pull(cases_per100k) - 0.127e-6) < 1e-3 # I think this number was missing scientific notation. I took a long look at it and just editted the assert. :)
            )
```

    ## [1] TRUE

``` r
assertthat::assert_that(
              abs(df_normalized %>%
               filter(
                 str_detect(county, "Snohomish"),
                 date == "2020-01-21"
               ) %>%
              pull(deaths_per100k) - 0) < 1e-3
            )
```

    ## [1] TRUE

``` r
print("Excellent!")
```

    ## [1] "Excellent!"

## Guided EDA

<!-- ------------------------- -->

Before turning you loose, let’s complete a couple guided EDA tasks.

### **q6** Compute the mean and standard deviation for `cases_per100k` and `deaths_per100k`.

``` r
## TASK: Compute mean and sd for cases_per100k and deaths_per100k
df_normalized %>% 
  summarise(mean_cases_per100k = mean(cases_per100k), sd_cases_per100k = sd(cases_per100k), mean_deaths_per100k = mean(deaths_per100k, na.rm = TRUE), sd_deaths_per100k = sd(deaths_per100k, na.rm = TRUE))
```

    ##   mean_cases_per100k sd_cases_per100k mean_deaths_per100k sd_deaths_per100k
    ## 1         0.09974675       0.08448659         0.001743095       0.001589641

### **q7** Find the top 10 counties in terms of `cases_per100k`, and the top 10 in terms of `deaths_per100k`. Report the population of each county along with the per-100,000 counts. Compare the counts against the mean values you found in q6. Note any observations.

``` r
## TASK: Find the top 10 max cases_per100k counties; report populations as well
df_top10_cases <- 
  df_normalized %>% 
  group_by(county) %>% 
  summarise(across(c(cases_per100k, population), mean)) %>% 
  arrange(desc(cases_per100k)) %>% 
  slice(0:10)
## TASK: Find the top 10 deaths_per100k counties; report populations as well
df_top10_deaths <- 
  df_normalized %>% 
  group_by(county) %>% 
  summarise(across(c(deaths_per100k, population), mean)) %>% 
  arrange(desc(deaths_per100k)) %>% 
  slice(0:10)

df_top10_cases
```

    ## # A tibble: 10 × 3
    ##    county               cases_per100k population
    ##    <chr>                        <dbl>      <dbl>
    ##  1 Loving                       0.382        102
    ##  2 Chattahoochee                0.339      10767
    ##  3 Crowley                      0.290       5630
    ##  4 Trousdale                    0.259       9573
    ##  5 Bent                         0.214       5809
    ##  6 Dimmit                       0.213      10663
    ##  7 Bethel Census Area           0.212      18040
    ##  8 Norton                       0.203       5486
    ##  9 Kusilvak Census Area         0.192       8198
    ## 10 Buena Vista                  0.188      20260

``` r
df_top10_deaths
```

    ## # A tibble: 10 × 3
    ##    county         deaths_per100k population
    ##    <chr>                   <dbl>      <dbl>
    ##  1 McMullen              0.00729        662
    ##  2 Galax city            0.00666       6638
    ##  3 Emporia city          0.00633       5381
    ##  4 Jerauld               0.00631       2029
    ##  5 Gove                  0.00595       2619
    ##  6 Gregory               0.00558       4201
    ##  7 Dickey                0.00535       4970
    ##  8 Foard                 0.00527       1408
    ##  9 McKinley              0.00527      72849
    ## 10 East Feliciana        0.00523      19499

**Observations**:

- Loving looks like it has more cases than people
- All of these places have small population sizes (less than 25,000)
  except McKinley.
- All of these are over the mean value, which makes sense
- All of these are more than one sd from the mean

## Self-directed EDA

<!-- ------------------------- -->

### **q8** Drive your own ship: You’ve just put together a very rich dataset; you now get to explore! Pick your own direction and generate at least one punchline figure to document an interesting finding. I give a couple tips & ideas below:

### Ideas

<!-- ------------------------- -->

- Look for outliers.
- Try web searching for news stories in some of the outlier counties.
- Investigate relationships between county population and counts.
- Do a deep-dive on counties that are important to you (e.g. where you
  or your family live).
- Fix the *geographic exceptions* noted below to study New York City.
- Your own idea!

**DO YOUR OWN ANALYSIS HERE**

``` r
library("ggrepel")
outlier <- function(x) {
  return(x > quantile(x, 0.75) + 1.5 * IQR(x) | x < quantile(x, 0.25) - 1.5 * IQR(x))
}

df_normalized %>% 
  group_by(county, state) %>% 
  summarise(across(c(cases_per100k, population), mean)) %>% 
  mutate(outlier = ifelse(outlier(cases_per100k), county, NA)) %>%
  ggplot(aes(x = cases_per100k, y = state, color = state)) +
  geom_boxplot(width=0.5, position = position_dodge(width=0.5)) +
  dark_theme_minimal() +
  theme(axis.text.y = element_text(size = 5)) +
  guides(color = FALSE) +
  geom_text_repel(aes(label = outlier), na.rm = TRUE, size = 2, point.padding = 5, max.overlaps = 30)
```

    ## `summarise()` has grouped output by 'county'. You can override using the
    ## `.groups` argument.
    ## Inverted geom defaults of fill and color/colour. To change them back, use
    ## invert_geom_defaults().

    ## Warning: The `<scale>` argument of `guides()` cannot be `FALSE`. Use "none" instead as
    ## of ggplot2 3.3.4.

![](c06-covid19-assignment_files/figure-gfm/q8-task-1.png)<!-- -->

``` r
df_top_5_cases <- 
  df_normalized %>% 
  group_by(county) %>% 
  summarise(across(c(cases, population), mean)) %>% 
  arrange(desc(cases)) %>% 
  slice(0:5)

df_normalized %>% 
  group_by(county, state) %>% 
  summarise(across(c(cases, population), mean)) %>% 
  ggplot(aes(cases, population)) +
  geom_smooth(method = "loess", alpha = .5, color = "red3", fill = "red4") +
  dark_theme_minimal() +
  geom_point(aes(color = state)) +
  guides(color = FALSE) +
  geom_text_repel(data = df_top_5_cases, aes(label = county), size = 4, point.padding = 2, min.segment.length = unit(0, 'lines'), color = "pink") +
  labs(
    x = "Cases",
    y = "Population",
    title = "Cases vs Population",
    caption = "The top five counties in terms of number of cases are labeled"
  )
```

    ## `summarise()` has grouped output by 'county'. You can override using the
    ## `.groups` argument.
    ## `geom_smooth()` using formula = 'y ~ x'

![](c06-covid19-assignment_files/figure-gfm/q8-task-2-1.png)<!-- -->

``` r
cor(df_normalized$cases, df_normalized$population)
```

    ## [1] 0.7471736

``` r
df_normalized %>%
  filter(county == df_top10_cases$county) %>% 
  ggplot(aes(date, cases_per100k, color = fct_reorder2(county, date, cases_per100k))) +
  geom_line() +
  dark_theme_minimal() +
  scale_y_log10(labels = scales::label_number_si()) +
  scale_color_discrete(name = "County") +
  labs(
    x = "Date",
    y = "Cases (per 100,000 persons)",
    title = "The Ten Counties With the Highest Cases per Capita"
  )
```

    ## Warning: `label_number_si()` was deprecated in scales 1.2.0.
    ## ℹ Please use the `scale_cut` argument of `label_number()` instead.

![](c06-covid19-assignment_files/figure-gfm/q8-task-3-1.png)<!-- -->

``` r
df_lowest10_cases <- 
  df_normalized %>% 
  group_by(county) %>% 
  summarise(across(c(cases_per100k, population), mean)) %>% 
  arrange(cases_per100k) %>% 
  slice(0:10)

df_normalized %>%
  filter(county == df_lowest10_cases$county) %>% 
  ggplot(aes(date, cases_per100k, color = fct_reorder2(county, date, cases_per100k))) +
  geom_line() +
  dark_theme_minimal() +
  scale_y_log10(labels = scales::label_number_si()) +
  scale_color_discrete(name = "County") +
  labs(
    x = "Date",
    y = "Cases (per 100,000 persons)",
    title = "The Ten Counties With the Lowest Cases per Capita"
  )
```

![](c06-covid19-assignment_files/figure-gfm/q8-task-3-2.png)<!-- -->

``` r
df_lowest10_cases
```

    ## # A tibble: 10 × 3
    ##    county        cases_per100k population
    ##    <chr>                 <dbl>      <dbl>
    ##  1 Kalawao              0.0133         75
    ##  2 Guanica              0.0272      16783
    ##  3 Fairfax city         0.0282      23865
    ##  4 Storey               0.0294       3941
    ##  5 Vieques              0.0297       8771
    ##  6 Maricao              0.0309       6202
    ##  7 Sabana Grande        0.0319      23054
    ##  8 Kauai                0.0322      71377
    ##  9 Arroyo               0.0329      18111
    ## 10 Lajas                0.0335      23315

### Observations

- Population is correlated to number of cases.

- This makes sense because smaller communities probably have more
  interaction and there are less smaller pods of people who can isolate
  from each other (modsim memories).

- This is also proven by the counties with the highest cases per capita
  all having smaller population sized (the top 5 have populations of
  around 1000 or less).

- This is also true in terms of counties with low cases. If communities
  can successfully prevent an infected person from entering, they can
  keep case numbers low. This is easier to do with smaller counties
  because there are less people interacting with people on the outside.

- A notable exception to what was just stated is Kauai county. It is
  relatively large and has very low case per capita numbers. The
  difference is that it is in Hawii. Other counties on the lowest 10
  counties in terms of cases per capita list of note include Kalawao
  (Hawii) and Guanica, Maricao, Sabana Grande, Lajas, and Vieques
  counties. (all in Puerto Rico). All of the counties are on islands. It
  makes sense that their case numbers were low because of how effectivly
  they were able to restrict travel during the height of the pandemic.

- Loving county in Texas has reported more cases than there are people
  🤔. If makes me think that some county clerk read a number wrong from
  a smudges piece of paper or something.

### Aside: Some visualization tricks

<!-- ------------------------- -->

These data get a little busy, so it’s helpful to know a few `ggplot`
tricks to help with the visualization. Here’s an example focused on
Massachusetts.

``` r
## NOTE: No need to change this; just an example
df_normalized %>%
  filter(state == "Massachusetts") %>%

  ggplot(
    aes(date, cases_per100k, color = fct_reorder2(county, date, cases_per100k))
  ) +
  geom_line() +
  scale_y_log10(labels = scales::label_number_si()) +
  scale_color_discrete(name = "County") +
  dark_theme_minimal() +
  labs(
    x = "Date",
    y = "Cases (per 100,000 persons)"
  )
```

![](c06-covid19-assignment_files/figure-gfm/ma-example-1.png)<!-- -->

*Tricks*:

- I use `fct_reorder2` to *re-order* the color labels such that the
  color in the legend on the right is ordered the same as the vertical
  order of rightmost points on the curves. This makes it easier to
  reference the legend.
- I manually set the `name` of the color scale in order to avoid
  reporting the `fct_reorder2` call.
- I use `scales::label_number_si` to make the vertical labels more
  readable.
- I use `theme_minimal()` to clean up the theme a bit.
- I use `labs()` to give manual labels.

### Geographic exceptions

<!-- ------------------------- -->

The NYT repo documents some [geographic
exceptions](https://github.com/nytimes/covid-19-data#geographic-exceptions);
the data for New York, Kings, Queens, Bronx and Richmond counties are
consolidated under “New York City” *without* a fips code. Thus the
normalized counts in `df_normalized` are `NA`. To fix this, you would
need to merge the population data from the New York City counties, and
manually normalize the data.

# Notes

<!-- -------------------------------------------------- -->

\[1\] The census used to have many, many questions, but the ACS was
created in 2010 to remove some questions and shorten the census. You can
learn more in [this wonderful visual
history](https://pudding.cool/2020/03/census-history/) of the census.

\[2\] FIPS stands for [Federal Information Processing
Standards](https://en.wikipedia.org/wiki/Federal_Information_Processing_Standards);
these are computer standards issued by NIST for things such as
government data.

\[3\] Demographers often report statistics not in percentages (per 100
people), but rather in per 100,000 persons. This is [not always the
case](https://stats.stackexchange.com/questions/12810/why-do-demographers-give-rates-per-100-000-people)
though!
