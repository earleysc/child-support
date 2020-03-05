The Baltimore Sun analyzed data on child support cases in the public
child support system (also known as IV-D cases) across Maryland during
the 2018 federal fiscal year provided by the Maryland Department of
Human Service in response to a public records request for a Feb. 26,
2020 story titled [“xxx”](xxx).

Here are the key data elements reported in the story. Note the raw data
is saved in the `input/` folder and was pre-processed using the
`cleaning.Rmd` script prior to analysis.

*Child support arrears debt by ZIP code*: - Several Baltimore city ZIP
codes owe millions in back child support, including one West Baltimore
ZIP code which owes $23 million in arrears. - In 10 city ZIP codes,
where about 15,000 parents collectively owe more than $233 million.

*Child support as welfare cost recovery*: - Maryland has about 16,500
cases set up for welfare cost recovery, almost half of them in
Baltimore. - Collectively in the state, non-custodial parents owe the
government $156 million in back child support to repay welfare benefits,
including $73 million in Baltimore.

*License suspensions by Maryland’s child support agency*: - Nearly
40,000 people had their driver’s license suspended by Maryland’s child
support agency as of September 2018. More than a third of those drivers
lived in Baltimore. - Some 1,900 professional and recreational licenses
had been suspended by the state, including 750 in Baltimore, in
September 2018. - Statewide, about 320 suspended professional licenses
were for certified medication technicians. Another 150 were rideshare
licenses. About 225 barbers and 120 certified nursing assistants had
their occupational privileges blocked.

*Child support debt among older parents*: - Among noncustodial parents
62 or older, two-thirds had back payments in September 2018.

### Load R libraries

``` r
library('tidyverse')
library('tidycensus')
library('sf')
```

### Read in the pre-processed caseload data

See the pre-processing code at `cleaning.Rmd` for detail on how the data
was cleaned prior to analysis.

``` r
caseload <- read_rds('output/caseload.rds') # statewide caseload
baci_caseload <- read_rds('output/baci_caseload.rds') # caseload for Baltimore city
```

### Child support arrears debt by ZIP code

\*Finding: Several Baltimore city ZIP codes owe millions in back child
support, including one West Baltimore ZIP code which owes $23 million in
arrears.

Finding: In 10 city ZIP codes, where about 15,000 parents collectively
owe more than $233 million.\*

Create a data frame, `baci_arrears.zips`, that groups the Baltimore city
caseload data by ZIP code to get the arrears that area owed by ZIP code.
Merge with `baci_caseload.ncps.zips`, which provides the number of
non-custodial parents in each city ZIP code that owe arrears.

``` r
baci_caseload.ncps.zips <- baci_caseload %>% filter(arrears_owed_total_september_2018 > 0) %>%
  select(ncp_zip, ncp_id) %>% 
  distinct() %>% 
  group_by(ncp_zip) %>%
  summarise(number_of_ncps = n()) 
  
baci_arrears.zips <- 
baci_caseload %>% group_by(ncp_zip) %>% 
  summarise(arrears = sum(arrears_owed_total_september_2018, 
                          na.rm = T)) %>% 
  arrange(desc(arrears)) %>% 
  ungroup() %>%
  mutate(cumulative = cumsum(arrears)) %>% 
  merge(baci_caseload.ncps.zips) %>%
  arrange(desc(arrears)) %>% 
  mutate(cumulative_ncps = cumsum(number_of_ncps))
```

Filter `baci_arrears.zips` to see the 10 ZIP codes that owe the most
arrears. As shown below, the arrears in these ZIP codes are 15+ million.

In West Baltimore [ZIP code
21216](https://www.google.com/maps/place/Baltimore,+MD+21216/@39.3108441,-76.6951786,14z/data=!3m1!4b1!4m5!3m4!1s0x89c81b6c23e6c4a9:0x777eeef9b68e99da!8m2!3d39.3096984!4d-76.6701475),
$23.4 million is owed in child support arrears.

``` r
baci_arrears.zips %>% head(10)
```

    ##    ncp_zip  arrears cumulative number_of_ncps cumulative_ncps
    ## 1    21215 33165588   33165588           2145            2145
    ## 2    21217 27896524   61062112           1706            3851
    ## 3    21213 26404097   87466208           1589            5440
    ## 4    21218 24143885  111610093           1509            6949
    ## 5    21229 23745388  135355481           1522            8471
    ## 6    21216 23414161  158769643           1504            9975
    ## 7    21223 20814035  179583678           1285           11260
    ## 8    21206 20239512  199823191           1464           12724
    ## 9    21207 18120026  217943216           1252           13976
    ## 10   21225 15561124  233504340           1036           15012

Use only state-owed arrears

``` r
baci_caseload.ncps.zips <- baci_caseload %>% filter(state_owed_arrears_total_september_2018 > 0) %>%
  select(ncp_zip, ncp_id) %>% 
  distinct() %>% 
  group_by(ncp_zip) %>%
  summarise(number_of_ncps = n()) 
  
baci_arrears.zips <- 
baci_caseload %>% group_by(ncp_zip) %>% 
  summarise(arrears = sum(state_owed_arrears_total_september_2018, 
                          na.rm = T)) %>% 
  arrange(desc(arrears)) %>% 
  ungroup() %>%
  mutate(cumulative = cumsum(arrears)) %>% 
  merge(baci_caseload.ncps.zips) %>%
  arrange(desc(arrears)) %>% 
  mutate(cumulative_ncps = cumsum(number_of_ncps))
```

<!-- ### Get poverty data by zip code -->
<!-- ```{r poverty} -->
<!-- v18.s <- load_variables(2018, "acs5/subject", cache = TRUE) -->
<!-- pov <- get_acs(geography = "zcta",  -->
<!--                  variables = c(poverty.rate = "S1701_C03_001"), -->
<!--                  year = 2018, -->
<!--                  output = 'wide') -->
<!-- ``` -->
<!-- ### Merge poverty, prison and arrears data -->
<!-- ```{r merge_map} -->
<!-- pov.merged <- merge(baci_arrears.zips,  -->
<!--                     pov,  -->
<!--                     by.x = 'ncp_zip', -->
<!--                     by.y = 'GEOID', -->
<!--                     all.x = T) -->
<!-- pov.prison.merged <- merge(pov.merged, -->
<!--                            md_prison, -->
<!--                            by.x = 'ncp_zip', -->
<!--                            by.y = 'zipcode_tabulation_areas', -->
<!--                            all.x = T) %>%  -->
<!--   mutate(arrears = arrears/1000000) -->
<!-- pov.prison.merged %>% filter(!is.na(poverty.rateE)) %>% write_csv('output/zipcodes_data_STATE.csv') -->
<!-- ``` -->
<!-- ### Check age of NCPs with arrears -->
<!-- ```{r} -->
<!-- caseload %>% filter(ncp_age >= 62 & arrears_owed_total_september_2018 > 0) %>% nrow() -->
<!-- caseload %>% filter(ncp_age >= 62) %>% nrow() -->
<!-- summary(caseload$ncp_age) -->
<!-- ``` -->
