# Purpose

The purpose of this folder is to practice using an application
programming interface for my econometrics tutorial.

``` r
rm(list = ls()) # Clean your environment:
gc() # garbage collection - It can be useful to call gc after a large object has been removed, as this may prompt R to return memory to the operating system.
```

    ##           used (Mb) gc trigger (Mb) limit (Mb) max used (Mb)
    ## Ncells  563277 30.1    1251505 66.9         NA   715714 38.3
    ## Vcells 1076081  8.3    8388608 64.0      16384  2010497 15.4

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.2.1     ✔ readr     2.2.0
    ## ✔ forcats   1.0.1     ✔ stringr   1.6.0
    ## ✔ ggplot2   4.0.2     ✔ tibble    3.3.1
    ## ✔ lubridate 1.9.5     ✔ tidyr     1.3.2
    ## ✔ purrr     1.2.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
if (!require("pacman", quietly = TRUE )) {
    install.packages("pacman")
    library(pacman)
}
if (!require("tibble", quietly = TRUE )) {
    install.packages("tibble")
    library(tibble)
}

if (!require("knitr", quietly = TRUE )) {
    install.packages("knitr")
    library(knitr)
}

p_load(wbstats,WDI,tidyverse,haven)

list.files('code/', full.names = T, recursive = T) %>% .[grepl('.R', .)] %>% as.list() %>% walk(~source(.))
```

# Setup

``` r
# Verify that the packages are ready (UNCOMMENT IF YOU WANT TO)
#library(help="WDI") # lists functions in the WDI package
#wbstats::wb_cachelist # glimpse of cached World Bank metadata
#WDI::WDI_data # explore WDI data structures
```

# Finding Indicator Codes

Each World Banking indicator has a unique alphanumeric code (e.g.,
NY.GDP.PCAP.KD). The code is the key that the API uses to retrieve the
correct seires. Both wbstats and WDI provide search function.

## Usng *wbstats* (Recommended)

``` r
# Search for indicators containing "GDP"
gdp_indicators <- wb_search("GDP")
head(gdp_indicators)
```

    ## # A tibble: 6 × 3
    ##   indicator_id       indicator                                    indicator_desc
    ##   <chr>              <chr>                                        <chr>         
    ## 1 5.51.01.10.gdp     Per capita GDP growth                        GDP per capit…
    ## 2 6.0.GDP_current    GDP (current $)                              GDP is the su…
    ## 3 6.0.GDP_growth     GDP growth (annual %)                        Annual percen…
    ## 4 6.0.GDP_usd        GDP (constant 2005 $)                        GDP is the su…
    ## 5 6.0.GDPpc_constant GDP per capita, PPP (constant 2011 internat… GDP per capit…
    ## 6 BG.GSR.NFSV.GD.ZS  Trade in services (% of GDP)                 Trade in serv…

``` r
# Narrow search to "GDP per capita "
gdp_pc <- wb_search("GDP per capita")
head(gdp_pc) #interactive table viewer in RStudio
```

    ## # A tibble: 6 × 3
    ##   indicator_id       indicator                                    indicator_desc
    ##   <chr>              <chr>                                        <chr>         
    ## 1 5.51.01.10.gdp     Per capita GDP growth                        GDP per capit…
    ## 2 6.0.GDPpc_constant GDP per capita, PPP (constant 2011 internat… GDP per capit…
    ## 3 NV.AGR.PCAP.KD.ZG  Real agricultural GDP per capita growth rat… The growth ra…
    ## 4 NY.GDP.PCAP.CD     GDP per capita (current US$)                 GDP per capit…
    ## 5 NY.GDP.PCAP.CN     GDP per capita (current LCU)                 GDP per capit…
    ## 6 NY.GDP.PCAP.KD     GDP per capita (constant 2010 US$)           GDP per capit…

``` r
# Important columns :
 # indicator _id -> code for wb_ data ()
 # indicator -> human - readable description
 # unit -> measurement unit (e.g., " constant 2015 US$")
```

## Using *WDI* (Classic)

``` r
# Search for indicators containing " inflation "
WDIsearch("inflation")
```

    ##                  indicator                                              name
    ## 11046       FP.CPI.TOTL.ZG             Inflation, consumer prices (annual %)
    ## 11048       FP.FPI.TOTL.ZG                 Inflation, food prices (annual %)
    ## 11050       FP.WPI.TOTL.ZG            Inflation, wholesale prices (annual %)
    ## 16183    NY.GDP.DEFL.87.ZG                Inflation, GDP deflator (annual %)
    ## 16184    NY.GDP.DEFL.KD.ZG                Inflation, GDP deflator (annual %)
    ## 16185 NY.GDP.DEFL.KD.ZG.AD Inflation, GDP deflator: linked series (annual %)

# Understanding Indicator Code Suffixes

The suffix of an indicator code encodes crucial information about the
unit, adjustment method, and base year. Misinterpreting suffixes is a
common source of errors in empirical work.

``` r
# Create the data
wb_data <- tribble(
  ~Suffix, ~`Meaning and Appropriate Use`,
  ".CD", "Current USD (nominal). Affected by both inflation and exchange rate movements.",
  ".KD", "Constant USD (real, inflation-adjusted). Base year is usually 2015.",
  ".PP.CD", "PPP-adjusted, current international dollars. Corrects for cross-country price levels.",
  ".PP.KD", "PPP-adjusted, constant international dollars. Combines inflation adjustment with PPP.",
  ".ZS", "Percentage share (ratio). Already unit-free (e.g., trade as % of GDP).",
  ".GD", "Share of GDP. Similar to .ZS but explicitly relative to GDP.",
  ".LN", "Natural logarithm of the underlying series. Used in internal calculations.",
  ".MA", "Moving average or smoothed series. Indicates values have been smoothed.",
  ".DT", "Decadal data (e.g., population census every 10 years).",
  ".AR", "Annual Report data. May differ from standard series due to reporting.",
  ".EST", "Estimated value (not directly reported). Used to fill gaps."
)

# Generate a Markdown-compatible table
kable(wb_data, format = "markdown", caption = "Table 1: World Bank Indicator Suffixes")
```

| Suffix | Meaning and Appropriate Use |
|:------|:----------------------------------------------------------------|
| .CD | Current USD (nominal). Affected by both inflation and exchange rate movements. |
| .KD | Constant USD (real, inflation-adjusted). Base year is usually 2015. |
| .PP.CD | PPP-adjusted, current international dollars. Corrects for cross-country price levels. |
| .PP.KD | PPP-adjusted, constant international dollars. Combines inflation adjustment with PPP. |
| .ZS | Percentage share (ratio). Already unit-free (e.g., trade as % of GDP). |
| .GD | Share of GDP. Similar to .ZS but explicitly relative to GDP. |
| .LN | Natural logarithm of the underlying series. Used in internal calculations. |
| .MA | Moving average or smoothed series. Indicates values have been smoothed. |
| .DT | Decadal data (e.g., population census every 10 years). |
| .AR | Annual Report data. May differ from standard series due to reporting. |
| .EST | Estimated value (not directly reported). Used to fill gaps. |

Table 1: World Bank Indicator Suffixes

**Economist’s rule of thumb**:

- Coss-Country level comparison -\> use .PP.CD or .PP.KD (PPP-adjusted).

- Growth over time within a country -\> use .KD (constant/real).

- Policy reports in nominal USD -\> .CD (current/nominal).

# Country Codes and Time Granularity

## ISO Country Code: ISO2 vs ISO3

The WB API accepts both two-letter and three-letter country codes:

- **ISO2**: Often used in plots and quick identification.

- **ISO3**: More informative and are the standard for merging with other
  international datasets(such as UN, IMF, or Penn Word Tables)

## Time Granularity in API Calls

The wbstats package automatically handles the format, but when
constructing raw API calls, you must specify dates correctly:

- **Annual data**: Use YYYY

- **Quarterly data**: Use YYYYQ (2020Q1)

- **Monthly data**: Use YYYM (2020M01)

# Downloading Data: Core Workflow

## Using wbstats

``` r
# Define indicators using a named vector ( names become column names )
my_indicators <- c(
gdp_per_capita = "NY.GDP.PCAP.KD", # constant 2015 USD
inflation = "FP.CPI.TOTL.ZG", # annual CPI inflation (%)
trade_openness = "NE.TRD.GNFS.ZS", # trade as % of GDP
school_enroll = "SE.SEC.ENRR" # secondary gross enrolment ratio
)

# Specify countries ( ISO2 or ISO3 codes )
 my_countries <- c("ZA","NG","KE","GH","EG")

 # Download data for 2000 -2023
africa_data <- wb_data(indicator = my_indicators, country = my_countries, start_date = 2000, end_date = 2023)

 # Inspect the structure
 head(africa_data)
```

    ## # A tibble: 6 × 8
    ##   iso2c iso3c country           date inflation trade_openness gdp_per_capita
    ##   <chr> <chr> <chr>            <dbl>     <dbl>          <dbl>          <dbl>
    ## 1 EG    EGY   Egypt, Arab Rep.  2000      2.68           39.0          2459.
    ## 2 EG    EGY   Egypt, Arab Rep.  2001      2.27           39.8          2492.
    ## 3 EG    EGY   Egypt, Arab Rep.  2002      2.74           41.0          2498.
    ## 4 EG    EGY   Egypt, Arab Rep.  2003      4.51           46.2          2525.
    ## 5 EG    EGY   Egypt, Arab Rep.  2004     11.3            57.8          2574.
    ## 6 EG    EGY   Egypt, Arab Rep.  2005      4.87           63.0          2636.
    ## # ℹ 1 more variable: school_enroll <dbl>

``` r
 glimpse(africa_data) # displays column types and units
```

    ## Rows: 120
    ## Columns: 8
    ## $ iso2c          <chr> "EG", "EG", "EG", "EG", "EG", "EG", "EG", "EG", "EG", "…
    ## $ iso3c          <chr> "EGY", "EGY", "EGY", "EGY", "EGY", "EGY", "EGY", "EGY",…
    ## $ country        <chr> "Egypt, Arab Rep.", "Egypt, Arab Rep.", "Egypt, Arab Re…
    ## $ date           <dbl> 2000, 2001, 2002, 2003, 2004, 2005, 2006, 2007, 2008, 2…
    ## $ inflation      <dbl> 2.683805, 2.269757, 2.737239, 4.507776, 11.270619, 4.86…
    ## $ trade_openness <dbl> 39.01794, 39.81043, 40.98707, 46.17964, 57.81991, 62.95…
    ## $ gdp_per_capita <dbl> 2458.608, 2492.034, 2498.481, 2524.805, 2574.423, 2635.…
    ## $ school_enroll  <dbl> 77.83395, 79.38974, 78.81741, 78.50661, 77.78418, NA, N…

## Filtering by Region or Income Group

``` r
# Retrieve country metadata
country_info <- wb_countries()

# Extract ISO3 codes for Sub - Saharan Africa
ssa_countries <- country_info %>%
filter(region == "Sub-Saharan Africa") %>%
pull(iso3c)

# Download GDP per capita for all SSA countries
ssa_data <- wb_data (indicator = c(gdp_pc = "NY.GDP.PCAP.KD"), country = ssa_countries, start_date = 2000, end_date = 2022)
# Approximately 48 countries 23 years = ~1 ,104 observations
```

## Understanding The Returned Dataframe

wbstats returns a tidy dataframe where each row corresponds to one
country-year observation. Key columns include:

``` r
# Summarise missing values across all columns
africa_data %>% summarise(across(everything(), ~ sum(is.na(.))))
```

    ## # A tibble: 1 × 8
    ##   iso2c iso3c country  date inflation trade_openness gdp_per_capita
    ##   <int> <int>   <int> <int>     <int>          <int>          <int>
    ## 1     0     0       0     0         0             24              0
    ## # ℹ 1 more variable: school_enroll <int>

``` r
# Note : Missing data often reflects non - reporting , which may correlate with
# institutional quality . Consider the implications for your analysis .
```
