AQI vs Mobility Project: Data Preparation
================
Anjelica Martinez
05/08/2020

-   [Introduction](#introduction)
-   [Data Sources](#data-sources)
    -   [Air Quality Data](#air-quality-data)
    -   [Mobility Data](#mobility-data)
    -   [Time Range Adjustment](#time-range-adjustment)
-   [Data Preparation](#data-preparation)
    -   [Mobility Data Prep](#mobility-data-prep)
    -   [AQI Data Prep](#aqi-data-prep)
    -   [Merge AQI and Mobility Dataframes](#merge-aqi-and-mobility-dataframes)

``` r
# Libraries
library(easypackages)
libraries("sjmisc", "tidyverse", "lubridate", "readr", "plyr")
```

Introduction
------------

The historic COVID-19 pandemic has caused nearly 50% of the world population to be subject to government “stay-at-home” and self-isolation orders. This sudden, drastic reduction in the mobility of the population has generated a natural experiment that tests the effect a near halt of mobility for individuals and drastic reduction of commercial operations would have on air quality. The metric used for this study is the mean daily mobility of individuals in a given US county.

Based on this criteria, we selected five major counties that had the highest level of mobility reduction as well as five other counties that had fewer reduction in mobility. A major county is defined as being in the top 40 most populous counties in the US, per the US Census Bureau.

The time period of this study is February 28 through March 27, the time during which some US counties enacted “shelter-in-place” and the self-isolation orders to control the community spread of COVID-19.

The five counties with the greatest reduction in citizen mobility issued “shelter-in-place” orders during this period.

The five counties with the least mobility reduction had no “shelter-in-place” orders during the studied time period and can serve as a control group. Our goal is to determine whether decrease in mobility correlates with increase in air quality and how strong of a relationship is there for these 10 counties.

**Counties of Interest**

Counties that decreased mobility **98-100%** between Feb. 28 and Mar. 27, 2020. [New York Times](https://www.nytimes.com/interactive/2020/04/02/us/coronavirus-social-distancing.html).

-   Arlington, VA
-   District of Columbia, DC
-   New York, NY
-   San Francisco, CA
-   Santa Clara, CA

Most populous counties that decreased mobility **30% or less** between between Feb. 28 and Mar. 27, 2020 [New York Times](https://www.nytimes.com/interactive/2020/04/02/us/coronavirus-social-distancing.html).

-   Miami-Dade, FL
-   Dallas, TX
-   Harris, TX
-   Maricopa, AZ
-   San Bernardino County, CA

Data Sources
------------

### Air Quality Data

We collected our air quality data from the [EPA's free API service](https://www.epa.gov/outdoor-air-quality-data/air-quality-index-daily-values-report). From the website, "The AirData Air Quality Index Summary Report displays an annual summary of Air Quality Index (AQI) values for counties or core based statistical areas (CBSA). Air Quality Index is an indicator of overall air quality, because it takes into account all of the criteria air pollutants measured within a geographic area. Although AQI includes all available pollutant measurements, you should be aware that many areas have monitoring stations for some, but not all, of the pollutants." You can read the documentation [here](https://www.epa.gov/outdoor-air-quality-data/about-air-data-reports).

### Mobility Data

We used the mobility statistics data collected by [Descarte Labs](https://github.com/descarteslabs/DL-COVID-19). The data represents "the distance a typical member of a given population moves in a day, at the US admin1 (state) and admin2 (county) level)". Follow the link to the github repository to read their documentation.

### Time Range Adjustment

Because the mobility dataset only has data from March 1st we decided to omit air quality data from the months of January and February. Thus, our analyses only covers from March 1st to April 18th.

Data Preparation
----------------

### Mobility Data Prep

First I cleaned and tidied up the *mobility* index csv file from Decartes Labs.

**Variables of Interest:**

-   `m50`: The median of the max-distance mobility for all samples in the specified region
-   `m50_index`: The percent of normal m50 in the region, with normal m50 defined during 2020-02-17 to 2020-03-07

If you would like to read more about the Descarte Labs mobility index, you can find the documentation [here](https://github.com/descarteslabs/DL-COVID-19).

``` r
# Import Messy Data
df <- read.csv("../Scripts/data/messy/us_mobility.csv", stringsAsFactors = FALSE)
(df <- as_tibble(df))
```

    ## # A tibble: 130,639 x 9
    ##    date     country_code admin_level admin1 admin2  fips samples   m50 m50_index
    ##    <chr>    <chr>              <int> <chr>  <chr>  <int>   <int> <dbl>     <int>
    ##  1 2020-03~ US                     1 Alaba~ ""         1  133826  8.33        79
    ##  2 2020-03~ US                     1 Alaba~ ""         1  143632 10.4         98
    ##  3 2020-03~ US                     1 Alaba~ ""         1  146009 10.5        100
    ##  4 2020-03~ US                     1 Alaba~ ""         1  149352 10.1         96
    ##  5 2020-03~ US                     1 Alaba~ ""         1  144109 11.0        104
    ##  6 2020-03~ US                     1 Alaba~ ""         1  141491 13.0        123
    ##  7 2020-03~ US                     1 Alaba~ ""         1  141163 11.4        107
    ##  8 2020-03~ US                     1 Alaba~ ""         1  138075  8.51        80
    ##  9 2020-03~ US                     1 Alaba~ ""         1  145037 10.9        103
    ## 10 2020-03~ US                     1 Alaba~ ""         1  143390 10.9        103
    ## # ... with 130,629 more rows

I renamed the columns to make it easier to understand what values they represented. I changed `admin1` to `state` and `admin2` to `county`. I then changed the date column from character format to data format.

``` r
# Rename columns
df <- dplyr::rename(df, state = admin1, county = admin2)

# Change character dates to R date format
df$date <- as.Date(df$date)

head(df)
```

    ## # A tibble: 6 x 9
    ##   date       country_code admin_level state county  fips samples   m50 m50_index
    ##   <date>     <chr>              <int> <chr> <chr>  <int>   <int> <dbl>     <int>
    ## 1 2020-03-01 US                     1 Alab~ ""         1  133826  8.33        79
    ## 2 2020-03-02 US                     1 Alab~ ""         1  143632 10.4         98
    ## 3 2020-03-03 US                     1 Alab~ ""         1  146009 10.5        100
    ## 4 2020-03-04 US                     1 Alab~ ""         1  149352 10.1         96
    ## 5 2020-03-05 US                     1 Alab~ ""         1  144109 11.0        104
    ## 6 2020-03-06 US                     1 Alab~ ""         1  141491 13.0        123

I created three dataframes for easier analysis:

-   `decrease`: mobility index values for counties that decreased mobility *98-100%* between Feb. 28 to Mar. 27
-   `increase`: mobility index values for counties that decreased mobility *30% or less* between Feb. 28 to Mar. 27
-   `df`: mobility index values for *all* counties

#### Decrease Dataframe

``` r
# Create dataframe that contains counties that decreased mobility 98-100% between Feb. 28 to Mar. 27
decrease <- df[(df$county %in% c("New York County", "Arlington County", "San Francisco County", "Santa Clara County")), ]
```

The mobility index had a peculiar entry. For Washington, D.C. they had the state listed as Washington D.C and the county column was left blank. I changed the state value to District of Columbia and added the District of Columbia value to all the District of Columbia state entries.

``` r
dc_data<- df[(df$state == "Washington, D.C."), ]
decrease <- bind_rows(decrease, dc_data)

# Replace Washington, DC with District of Columbia
decrease$state[decrease$state == "Washington, D.C."] <- "District of Columbia"

# Add District of Columbia value
decrease[197:245, 5] <- "District of Columbia"

head(decrease)
```

    ## # A tibble: 6 x 9
    ##   date       country_code admin_level state county  fips samples   m50 m50_index
    ##   <date>     <chr>              <int> <chr> <chr>  <int>   <int> <dbl>     <int>
    ## 1 2020-03-01 US                     2 Cali~ San F~  6075    4513  2.66        77
    ## 2 2020-03-02 US                     2 Cali~ San F~  6075    4139  3.34        97
    ## 3 2020-03-03 US                     2 Cali~ San F~  6075    5251  3.28        95
    ## 4 2020-03-04 US                     2 Cali~ San F~  6075    5503  3.46       101
    ## 5 2020-03-05 US                     2 Cali~ San F~  6075    4693  3.42       100
    ## 6 2020-03-06 US                     2 Cali~ San F~  6075    4683  3.84       112

Because the AQI dataframe excludes the county suffix, I removed the 'county' suffix from all the country entries for cohesiveness and easier analysis.

``` r
# Remove County from county name
decrease$county <- gsub(" County", "", decrease$county)

# Check that District of Columbia values have been added
decrease[(decrease$county == "District of Columbia"), ]
```

    ## # A tibble: 49 x 9
    ##    date       country_code admin_level state county  fips samples   m50
    ##    <date>     <chr>              <int> <chr> <chr>  <int>   <int> <dbl>
    ##  1 2020-03-01 US                     1 Dist~ Distr~    11    4735  3.74
    ##  2 2020-03-02 US                     1 Dist~ Distr~    11    4344  2.80
    ##  3 2020-03-03 US                     1 Dist~ Distr~    11    5165  3.58
    ##  4 2020-03-04 US                     1 Dist~ Distr~    11    5349  3.66
    ##  5 2020-03-05 US                     1 Dist~ Distr~    11    4956  3.90
    ##  6 2020-03-06 US                     1 Dist~ Distr~    11    4874  4.27
    ##  7 2020-03-07 US                     1 Dist~ Distr~    11    4928  3.97
    ##  8 2020-03-08 US                     1 Dist~ Distr~    11    5380  4.51
    ##  9 2020-03-09 US                     1 Dist~ Distr~    11    5199  3.38
    ## 10 2020-03-10 US                     1 Dist~ Distr~    11    5090  3.52
    ## # ... with 39 more rows, and 1 more variable: m50_index <int>

``` r
# Check decrease dataframe
head(decrease)
```

    ## # A tibble: 6 x 9
    ##   date       country_code admin_level state county  fips samples   m50 m50_index
    ##   <date>     <chr>              <int> <chr> <chr>  <int>   <int> <dbl>     <int>
    ## 1 2020-03-01 US                     2 Cali~ San F~  6075    4513  2.66        77
    ## 2 2020-03-02 US                     2 Cali~ San F~  6075    4139  3.34        97
    ## 3 2020-03-03 US                     2 Cali~ San F~  6075    5251  3.28        95
    ## 4 2020-03-04 US                     2 Cali~ San F~  6075    5503  3.46       101
    ## 5 2020-03-05 US                     2 Cali~ San F~  6075    4693  3.42       100
    ## 6 2020-03-06 US                     2 Cali~ San F~  6075    4683  3.84       112

#### Increase Dataframe

This dataframe contains the mobility index values for counties that decreased mobility *30% or less* between Feb. 28 to Mar. 27.

``` r
# Create dataframe that contains Counties that decreased mobility 30% or less between Feb. 28 to Mar. 27
increase <- df[(df$county %in% c("Miami-Dade County", "Dallas County", "Harris County", "Maricopa County", "San Bernardino County")), ]
```

This dataset contained several states that shared the same counties names. Here, I removed the states that we were not analyzing as well as the county suffix.

``` r
increase <- filter(increase, !(state %in% c("Alabama", "Arkansas", "Iowa", "Missouri", "Georgia")))

# Remove "county" from county name
increase$county <- gsub(" County","", increase$county)
```

I then combined these two datasets together to form `sample_df` that only contained the data for the 10 counties of interest. I kept the original intact dataframe as `df` in case we decided to include more counties.

``` r
# Create df with counties of interest (our sample)
sample_df <- bind_rows(decrease, increase)
```

I decided to rename the variables when I saved the .Rdata file in order to make it easier identification in future analyses scripts.

``` r
# Rename with clearer variables
all_mobility_data <- df
decreased_mobility <- decrease
increased_mobility <- increase
all_selected_counties <- sample_df

# Check variables
head(all_mobility_data)
```

    ## # A tibble: 6 x 9
    ##   date       country_code admin_level state county  fips samples   m50 m50_index
    ##   <date>     <chr>              <int> <chr> <chr>  <int>   <int> <dbl>     <int>
    ## 1 2020-03-01 US                     1 Alab~ ""         1  133826  8.33        79
    ## 2 2020-03-02 US                     1 Alab~ ""         1  143632 10.4         98
    ## 3 2020-03-03 US                     1 Alab~ ""         1  146009 10.5        100
    ## 4 2020-03-04 US                     1 Alab~ ""         1  149352 10.1         96
    ## 5 2020-03-05 US                     1 Alab~ ""         1  144109 11.0        104
    ## 6 2020-03-06 US                     1 Alab~ ""         1  141491 13.0        123

``` r
head(decreased_mobility)
```

    ## # A tibble: 6 x 9
    ##   date       country_code admin_level state county  fips samples   m50 m50_index
    ##   <date>     <chr>              <int> <chr> <chr>  <int>   <int> <dbl>     <int>
    ## 1 2020-03-01 US                     2 Cali~ San F~  6075    4513  2.66        77
    ## 2 2020-03-02 US                     2 Cali~ San F~  6075    4139  3.34        97
    ## 3 2020-03-03 US                     2 Cali~ San F~  6075    5251  3.28        95
    ## 4 2020-03-04 US                     2 Cali~ San F~  6075    5503  3.46       101
    ## 5 2020-03-05 US                     2 Cali~ San F~  6075    4693  3.42       100
    ## 6 2020-03-06 US                     2 Cali~ San F~  6075    4683  3.84       112

``` r
head(increased_mobility)
```

    ## # A tibble: 6 x 9
    ##   date       country_code admin_level state county  fips samples   m50 m50_index
    ##   <date>     <chr>              <int> <chr> <chr>  <int>   <int> <dbl>     <int>
    ## 1 2020-03-01 US                     2 Ariz~ Maric~  4013   78925  5.39        62
    ## 2 2020-03-02 US                     2 Ariz~ Maric~  4013   82514  8.44        97
    ## 3 2020-03-03 US                     2 Ariz~ Maric~  4013   84011  8.55        98
    ## 4 2020-03-04 US                     2 Ariz~ Maric~  4013   85357  8.67       100
    ## 5 2020-03-05 US                     2 Ariz~ Maric~  4013   82941  9.14       105
    ## 6 2020-03-06 US                     2 Ariz~ Maric~  4013   81537  9.71       111

``` r
head(all_selected_counties)
```

    ## # A tibble: 6 x 9
    ##   date       country_code admin_level state county  fips samples   m50 m50_index
    ##   <date>     <chr>              <int> <chr> <chr>  <int>   <int> <dbl>     <int>
    ## 1 2020-03-01 US                     2 Cali~ San F~  6075    4513  2.66        77
    ## 2 2020-03-02 US                     2 Cali~ San F~  6075    4139  3.34        97
    ## 3 2020-03-03 US                     2 Cali~ San F~  6075    5251  3.28        95
    ## 4 2020-03-04 US                     2 Cali~ San F~  6075    5503  3.46       101
    ## 5 2020-03-05 US                     2 Cali~ San F~  6075    4693  3.42       100
    ## 6 2020-03-06 US                     2 Cali~ San F~  6075    4683  3.84       112

``` r
# Save as Rdata file
# save(decreased_mobility, increased_mobility, all_mobility_data, all_selected_counties, file = "mobility.RData")

# Check that data loads
# load("mobility.RData")
```

### AQI Data Prep

**Variables of Interest:**

-   `AQI`: average air quality index for that day
-   `Pollutant names (e.g. Ozone, PM25)`: each have their average daily value
-   `main_pollutant`: from the original data, represented which pollutant was the main pollutant affecting AQI

The API system provided by EPA required us to download each counties daily average aqi and pollutant values individually. I used an apply function (loop) to quickly process all the csv files and combine them into one dataset called `df`.

``` r
# Apply loop to import csv files at once
files <- list.files(path = "../Scripts/data/messy/daily_aqi_values", pattern = "*.csv", full.names = T)
df <- ldply(files, read_csv)

# Change to data format
df$Date <- as.Date(df$Date)

# Rearrange columns
df <- move_columns(df, County, .after = "Main Pollutant")

#Change to tibble format
df <- as_tibble(df)

head(df)
```

    ## # A tibble: 6 x 13
    ##   Date       `Overall AQI Va~ `Main Pollutant` County `Site Name (of ~
    ##   <date>                <dbl> <chr>            <chr>  <chr>           
    ## 1 0001-01-20               27 PM2.5            Arlin~ Aurora Hills Vi~
    ## 2 0001-04-20               30 PM2.5            Arlin~ Aurora Hills Vi~
    ## 3 0001-07-20               30 PM2.5            Arlin~ Aurora Hills Vi~
    ## 4 0001-10-20               21 PM2.5            Arlin~ Aurora Hills Vi~
    ## 5 NA                       29 PM2.5            Arlin~ Aurora Hills Vi~
    ## 6 NA                       18 PM2.5            Arlin~ Aurora Hills Vi~
    ## # ... with 8 more variables: `Site ID (of Overall AQI)` <chr>, `Source (of
    ## #   Overall AQI)` <chr>, Ozone <chr>, PM25 <chr>, CO <chr>, SO2 <chr>,
    ## #   PM10 <chr>, NO2 <chr>

Next I adjusted the columns to tidy format (lowercase, minimal) and renamed `Overall AQI` to `aqi`.

``` r
# Rename Columns
names(df)
```

    ##  [1] "Date"                       "Overall AQI Value"         
    ##  [3] "Main Pollutant"             "County"                    
    ##  [5] "Site Name (of Overall AQI)" "Site ID (of Overall AQI)"  
    ##  [7] "Source (of Overall AQI)"    "Ozone"                     
    ##  [9] "PM25"                       "CO"                        
    ## [11] "SO2"                        "PM10"                      
    ## [13] "NO2"

``` r
(names(df) <- c("date", "aqi", "main_pollutant",
               "county", "site_name", "site_id",
               "source", "ozone", "pm25", "co",
               "so2", "pm10", "no2"
               ))
```

    ##  [1] "date"           "aqi"            "main_pollutant" "county"        
    ##  [5] "site_name"      "site_id"        "source"         "ozone"         
    ##  [9] "pm25"           "co"             "so2"            "pm10"          
    ## [13] "no2"

Before saving the .Rdata file I renamed the df variable to aqi\_df so that when it was imported it would not replace another variable called df.

``` r
# rename variable
aqi_df <- df

# Save Workspace
# save(aqi_df, file = "daily_aqi_and_pollutants.RData")
```

### Merge AQI and Mobility Dataframes

``` r
# Load .Rdata files
load("../Scripts/data/tidy/mobility.RData")
load("../Scripts/data/tidy/daily_aqi_and_pollutants.RData")

# Create mobility dataframe with selected values
mobility_df <- all_selected_counties %>%
  select(date, county, m50, m50_index)

# Create aqi dataframe with selected values
aqi_df <- aqi_df %>%
  filter(date >= "2020/03/01") %>%
  select(date, county, aqi, main_pollutant, ozone, pm25, co, so2, pm10, no2)

# Combine dataframes together and match by date and county
df <- merge(mobility_df, aqi_df, by = c("date", "county"))
```

Next, I seperated the counties by mobility percentage as we did previously but this time with mobility and aqi values included. Finally, I saved it as a the final .Rdata file to use for our analyses.

``` r
# counties that decreased by <30%
high_mobility <- df[(df$county %in% c("New York", "Arlington", "San Francisco", "Santa Clara", "District of Columbia")), ]

# counties that decreased by 98-100%
low_mobility <- df[(df$county %in% c("Miami-Dade", "Dallas", "Harris", "Maricopa", "San Bernardino")), ]

# save the final dataset to use for analysis
# save(df, low_mobility, high_mobility, file = "data/tidy/tidy_mobility_vs_aqi.RData")

head(df)
```

    ##         date               county   m50 m50_index aqi main_pollutant ozone pm25
    ## 1 2020-03-01            Arlington 3.270        71  37          Ozone    37    .
    ## 2 2020-03-01 District of Columbia 3.739       102  39          Ozone    39   26
    ## 3 2020-03-01             Maricopa 5.387        62  44          Ozone    44   43
    ## 4 2020-03-01           Miami-Dade 5.467        68  47          Ozone    47   30
    ## 5 2020-03-01             New York 0.462        23  35          Ozone    35    .
    ## 6 2020-03-01       San Bernardino 4.508        57  47          PM2.5    47   47
    ##     co  so2 pm10  no2
    ## 1 <NA> <NA> <NA> <NA>
    ## 2 <NA> <NA> <NA> <NA>
    ## 3    7    1   17   29
    ## 4 <NA> <NA> <NA> <NA>
    ## 5    2 <NA>    . <NA>
    ## 6 <NA> <NA> <NA> <NA>

``` r
head(high_mobility)
```

    ##         date               county   m50 m50_index aqi main_pollutant ozone pm25
    ## 1 2020-03-01            Arlington 3.270        71  37          Ozone    37    .
    ## 2 2020-03-01 District of Columbia 3.739       102  39          Ozone    39   26
    ## 5 2020-03-01             New York 0.462        23  35          Ozone    35    .
    ## 7 2020-03-01        San Francisco 2.663        77  36          Ozone    36   20
    ## 8 2020-03-01          Santa Clara 4.088        57  36          Ozone    36   29
    ## 9 2020-03-02            Arlington 4.594       100  44          Ozone    44    .
    ##     co  so2 pm10  no2
    ## 1 <NA> <NA> <NA> <NA>
    ## 2 <NA> <NA> <NA> <NA>
    ## 5    2 <NA>    . <NA>
    ## 7 <NA> <NA> <NA> <NA>
    ## 8 <NA> <NA> <NA> <NA>
    ## 9 <NA> <NA> <NA> <NA>

``` r
head(low_mobility)
```

    ##          date         county   m50 m50_index aqi main_pollutant ozone pm25   co
    ## 3  2020-03-01       Maricopa 5.387        62  44          Ozone    44   43    7
    ## 4  2020-03-01     Miami-Dade 5.467        68  47          Ozone    47   30 <NA>
    ## 6  2020-03-01 San Bernardino 4.508        57  47          PM2.5    47   47 <NA>
    ## 10 2020-03-02         Dallas 9.311       100  50          PM2.5    23   50 <NA>
    ## 12 2020-03-02         Harris 8.717        97  48          PM2.5    37   48 <NA>
    ## 13 2020-03-02       Maricopa 8.438        97  41          Ozone    41   26    5
    ##     so2 pm10  no2
    ## 3     1   17   29
    ## 4  <NA> <NA> <NA>
    ## 6  <NA> <NA> <NA>
    ## 10 <NA> <NA> <NA>
    ## 12 <NA> <NA> <NA>
    ## 13    1   19   24
