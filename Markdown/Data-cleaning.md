Data cleaning for the raw N and More participants data
================

## Overview

This code cleans the raw N and More participants data:

1.  convert data from wide to long format
2.  delete NA items
3.  create a separate subj level accuracy column and use it to
    categorize children into 4 quartiles
4.  adding item feature information
5.  output the results into new csv files for further analysis

It also identifies children who had completed both tasks, by
concatenating their “DOB, DOE, and location” and find common values both
the n and more data.

## Load libraries

``` r
library("here")
library("tidyverse")
```

## For the N task

``` r
# load input data
data_subj = read.csv(here("Data","data_n_raw_wide.csv"), check.names = FALSE) 
data_item = read.csv(here("Data","data_n_item.csv"), check.names = FALSE)

# cleaning
data_cleaned = data_subj %>%
  gather(key = item, value = acc, c(13:43)) %>% # convert from wide to long format
  mutate(SES = ifelse(ses == "#N/A", "NA", ses), # standardize the SES values
         SEX = ifelse(is.na(sex), "NA", sex)) %>% 
  filter(!is.na(acc)) # delete NA items

# create quartile group column
data_quartile = data_subj %>%
  group_by(id) %>%
  summarise(subj_acc = mean(acc)) %>%
  mutate(quartile = ntile(subj_acc,4))

# adding quartile and item feature information to the raw data
data_cleaned = data_cleaned %>%
  left_join(data_quartile, by = "id") %>%
  left_join(data_item, by = "item") %>%
  mutate(within_id = paste(dob, doe, sex, location)) # create a column for within subject id

# output the cleaned data
write.csv(data_cleaned, here("Data/data_n_long.csv"), row.names = FALSE)
```

## For the More task

``` r
# load input data
data_subj = read.csv(here("Data","data_more_raw_wide.csv"), check.names = FALSE) 
data_item = read.csv(here("Data","data_more_item.csv"), check.names = FALSE)

# cleaning
data_cleaned = data_subj %>%
  gather(key = item, value = acc, c(13:92)) %>% # convert from wide to long format
  mutate(SES = ifelse(ses == "#N/A", "NA", ses), # standardize the SES values
         SEX = ifelse(is.na(sex), "NA", sex)) %>% 
  filter(!is.na(acc)) # delete NA items

# create quartile group column
data_quartile = data_subj %>%
  group_by(id) %>%
  summarise(subj_acc = mean(acc)) %>%
  mutate(quartile = ntile(subj_acc,4))

# adding quartile and item feature information to the raw data
data_cleaned = data_cleaned %>%
  left_join(data_quartile, by = "id") %>%
  left_join(data_item, by = "item") %>%
  mutate(within_id = paste(dob, doe, sex, location)) # create a column for within subject id

# output the cleaned data
write.csv(data_cleaned, here("Data/data_more_long.csv"), row.names = FALSE)
```

## Identify children who had both tasks

``` r
data_more_long = read.csv(here("Data/data_more_long.csv"), check.names = FALSE) %>%
  distinct(within_id)

data_n_long = read.csv(here("Data/data_n_long.csv"), check.names = FALSE) %>%
  distinct(within_id) # there were twins in the which n data, but we can not decide which more task goes with which twin, so delete all 5 sets of twins. 

all_within_ids = inner_join(data_more_long, data_n_long, by = "within_id", all = TRUE) # find children who did both tasks

# output these ids in a csv file
write.csv(all_within_ids, here("Data/data_within_ids.csv"), row.names = F)
```
